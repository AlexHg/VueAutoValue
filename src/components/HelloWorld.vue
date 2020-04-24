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
      let adjustable_overflow = 0;

      virtual_grid.map((gridElement, key) => {
        let temporal_current_percent = global_current_percent;
        temporal_current_percent += parseInt(gridElement.percent, 0);
        console.log(temporal_current_percent, this.global_max_percent);

        let overflow = temporal_current_percent - this.global_max_percent;
        if (
          temporal_current_percent > this.global_max_percent &&
          !this.auto_adjust
        ) {
          console.log(
            `Te pasaste por ${overflow}, Ajustando este ultimo valor!`
          );
          gridElement.percent =
            this.global_max_percent - global_current_percent;
        } else {
          if (
            temporal_current_percent > this.global_max_percent &&
            this.auto_adjust
          ) {
            console.log(
              `Te pasaste por ${overflow}, Ajustando todos los valores a conveniencia de este ultimo valor!`
            );
            adjustable_overflow = overflow;
          }
          global_current_percent = temporal_current_percent;
        }
        return gridElement;
      });
      virtual_grid[index].percent = virtual_grid.pop().percent;

      if (this.auto_adjust && adjustable_overflow > 0) {
        this.adjustOverflow(adjustable_overflow, index);
      }
      console.log("_________");
      return virtual_grid[index];
    },
    adjustOverflow(overflow, to_exclude_index) {
      let unit_adjust = Math.floor(
        overflow /
          this.grid.filter(ge => ge.percent > 0 && ge.id !== to_exclude_index)
            .length
      );
      let mod_adjust =
        overflow %
        this.grid.filter(ge => ge.percent > 0 && ge.id !== to_exclude_index)
          .length;

      console.log(
        "Adjusting percents: " + unit_adjust + ", mod: " + mod_adjust
      );
      this.grid
        .filter(ge => ge.percent > 0 && ge.id !== to_exclude_index)
        .map((gridElement, index) => {
          gridElement.percent -= unit_adjust;
          if (index < mod_adjust) gridElement.percent--;
          return gridElement;
        });
    }
  },
  data() {
    return {
      global_max_percent: 100,
      auto_adjust: true,
      proto: {
        id: 0,
        percent: 0
      },
      grid: []
    };
  }
};
</script>

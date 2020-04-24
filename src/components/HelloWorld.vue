<template>
  <div class="hello">
    <input type="checkbox" v-model="auto_adjust" id="auto_adjust">
    <label for="auto_adjust">Auto ajustar valores anteriores</label>
    <br>
    <button @click="addGridElement">Agregar</button>
    <br>

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

    // Metodo de validación: Valida las unidades del catalogo de gridElement en busca del elemento que no cumpla las reglas establecidas
    validatePercent(index) {
      // Se verifica que el valor introducido no supere el maximo establecido
      if (this.grid[index].percent > this.global_max_percent)
        this.grid[index].percent = this.global_max_percent;

      // Genera un grid virtual para realizar calculos
      let virtual_grid = [...this.grid];

      // Genera un clon del gridElement a validar
      let virtual_grid_selected_element = { ...virtual_grid[index] };

      // Obtiene el gridElement y lo separa del grid llevandolo al final
      virtual_grid[index].percent = 0;
      virtual_grid.push(virtual_grid_selected_element);

      // Se declara el porcentaje global Y el desboramiento a ajustar en sus valores iniciales
      let global_current_percent = 0;
      let adjustable_overflow = 0;

      // Recorre el grid virtual en busca de elementos que no cumplan con las reglas
      virtual_grid.map((gridElement, key) => {
        // Crea una simulacion del porcentaje
        let temporal_current_percent = global_current_percent;
        temporal_current_percent += parseInt(gridElement.percent, 0);

        //console.log(temporal_current_percent, this.global_max_percent);

        // Calculo de desbordamiento / deuda
        let overflow = temporal_current_percent - this.global_max_percent;
        // Proceso de selección de ajuste:
        if (
          temporal_current_percent > this.global_max_percent &&
          !this.auto_adjust
        ) {
          // Indicador de tipo de ajuste
          //console.log(
          // `Te pasaste por ${overflow}, Ajustando este ultimo valor!`
          //);

          // Metodo de ajuste por bloqueo: Se calcula el porcentaje restante global y se bloquea el elemento seleccionado para no exceder el limite
          this.adjustLockflow(global_current_percent, index);
        } else {
          if (
            temporal_current_percent > this.global_max_percent &&
            this.auto_adjust
          ) {
            // Indicador de tipo de ajuste
            //console.log(
            //  `Te pasaste por ${overflow}, Ajustando todos los valores a conveniencia de este ultimo valor!`
            //);

            // Se asigna un desbordamiento
            adjustable_overflow = overflow;
          }

          // Se actualiza el porcentaje actual global
          global_current_percent = temporal_current_percent;
        }
        return gridElement;
      });
      // Se asigna el nuevo valor al gridElement estudiado
      virtual_grid[index].percent = virtual_grid.pop().percent;

      // Si encuentra desbordamiento ajusta los valores
      if (this.auto_adjust && adjustable_overflow > 0) {
        this.adjustOverflow(adjustable_overflow, index, this.adjust_type);
      }

      //console.log("_________");
      return virtual_grid[index];
    },

    // Metodo de ajuste por bloqueo <<Basic Method>>
    adjustLockflow(global_current_percent, to_lock_index) {
      // Reduce el valor del gridElement estudiado al maximo permitido
      this.grid[to_lock_index] =
        this.global_max_percent - global_current_percent;
    },

    // Metodo de ajuste por deudas / desbordamiento
    adjustOverflow(overflow, to_exclude_index) {
      // Elementos ajustables: Elementos a los que sus caracteristicas les permiten ser ajustados
      let adjustable_grid_elements = this.grid.filter(
        ge => ge.percent > 0 && ge.id !== to_exclude_index
      );

      // Si el desbordamiento/Deuda desaparece termina el ciclo
      if (overflow === 0) return;

      // Ajuste unitario: Ajuste aplicado a cada elemento al que aplica
      let unit_adjust = Math.floor(overflow / adjustable_grid_elements.length);
      // Ajuste modular: Ajuste repartido entre cada elemento al que aplica
      let mod_adjust = overflow % adjustable_grid_elements.length;
      // Deuda: Deuda generada a partir de los valores negativas dentro de la 'negociación'
      let debt = 0;

      // Indicador de ajuste
      //console.log(
      //  "Adjusting percents: " + unit_adjust + ", mod: " + mod_adjust
      //);

      // Negociación: Contexto en el que se decide la repartición de ajuste (-) <<Trusty Method>>
      if (this.adjust_type === "Equal")
        this.grid
          .filter(
            gridElement =>
              gridElement.percent > 0 && gridElement.id !== to_exclude_index
          )
          .map((gridElement, index) => {
            // Se calcula por partes iguales la cantidad de desbordamiento y se ajustan
            gridElement.percent -= unit_adjust;

            if (index < mod_adjust) gridElement.percent--;
            // Se identifican los elementos que sobrepasan su limite ajustable
            if (gridElement.percent < 0) {
              // Los elementos con porcentajes negativos generan deuda en el sistema
              debt += gridElement.percent;
              gridElement.percent = 0;
            }
            return gridElement;
          });

      // Negociación calculada por porcentajes: no esta lista
      if (this.adjust_type === "Fair")
        this.grid
          .filter(gridElement => gridElement.id !== to_exclude_index)
          .map((gridElement, index) => {
            // Calcula el nuevo porcentaje maximo que será tomado como el 100% para todos
            let new_max_percent = this.global_max_percent - gridElement.percent;

            // Se calcula por partes iguales la cantidad de desbordamiento y se ajustan
            gridElement.percent =
              (parseInt(gridElement.percent, 0) / this.global_max_percent) *
              new_max_percent;

            return gridElement;
          });

      // Tras la primer negociación; la deuda generada requiere que se redistribuyan los porcentajes entre los posibles repitiendo el proceso
      this.adjustOverflow(debt, to_exclude_index);
    }
  },
  data() {
    return {
      // Puntaje maximo de porcentaje en el sistema
      global_max_percent: 100,
      // Permisos de auto-ajuste
      auto_adjust: false,
      adjust_type: "Equal", // Equal || Fair
      // Prototipo de gridElement
      proto: {
        id: 0,
        percent: 0
      },
      // Grilla
      grid: []
    };
  }
};
</script>

<template>
  <div>
    <label for="roles" >Aula o carrico que quieres reservar </label>
    <select id="roles" name="roles" aria-expanded="false">
      <option v-for="recurso in recursos" :key="recurso">{{recurso}}</option>
    </select>
  </div>
  <div>
    <table border="1">
      <thead>
        <tr>
          <th>Hora/Día</th>
          <th v-for="day in days" :key="day">{{ day }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="hour in hours" :key="hour">
          <td>{{ hour }}</td>
          <td
            v-for="day in days"
            :key="day"
            class="cell"
            @click="selectSlot(day, hour)">
            <div v-if="reservations[day]?.[hour]">
              {{ reservations[day][hour].name }} ({{ reservations[day][hour].students }})
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h3>{{currentUser}}</h3>
        <h3>¿Quieres reservar para {{ selectedSlot.day }} a las {{ selectedSlot.hour }}?</h3>
        <input
          type="number"
          v-model.number="students"
          placeholder="Número de alumnos"
        />
        <button @click="confirmReservation">Confirmar</button>
        <button @click="closeModal">Cancelar</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
table {
  margin-top: 3%;
  border-collapse: collapse;
  text-align: center;
  margin-left: auto;
  margin-right: auto;
  width: 90vw;
  font-size: 20px;
}
.cell {
  height: 60px;
  cursor: pointer;
}
.cell:hover {
  background-color: #f0f0f0;
}
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.3);
}
</style>

<script>
export default {
  data() {
    return {
      days: ["Lunes", "Martes", "Miércoles", "Jueves", "Viernes"],
      hours: ["08:00", "09:00", "10:00", "11:30", "12:30", "13:30"],
      recursos: ["Biblioteca", "Aula 2.1", "Aula 0.9", "Carrito 1ª Planta", "Carrito 2ª Planta"],
      reservations: {}, // Objeto para almacenar reservas
      showModal: false, // Controla si se muestra el modal
      selectedSlot: { day: "", hour: "" }, // Casilla seleccionada
      students: 0, // Número de alumnos
      currentUser: "Usuario1", // Simulación del usuario logueado
    };
  },
  methods: {
    selectSlot(day, hour) {
      this.selectedSlot = { day, hour };
      this.students = 0; // Reinicia la cantidad
      this.showModal = true; // Muestra el modal
    },
    confirmReservation() {
      const { day, hour } = this.selectedSlot;
      if (!this.reservations[day]) {
        this.$set(this.reservations, day, {});
      }
      this.$set(this.reservations[day], hour, {
        name: this.currentUser,
        students: this.students,
      });
      this.closeModal();
    },
    closeModal() {
      this.showModal = false;
    },
  },
};
</script>

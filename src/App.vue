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
          <th v-for="dayId in daysId" :key="dayId">{{ getDaysName(dayId) }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="hourId in hoursId" :key="hourId">
          <td>{{ getTimeSlot(hourId) }}</td>
          <td
            v-for="dayId in daysId"
            :key="dayId"
            class="cell"
            @click="selectSlot(day, hourId)">

            <div v-if="reservations[day]?.[hourId]">
              {{ reservations[day][hourId].name }} ({{ reservations[day][hourId].students }})
              <button class="delete-btn" @click.stop="deleteReservation(day, hourId)">X</button>
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
.delete-btn {
  background-color: red;
  color: white;
  border: none;
  border-radius: 4px;
  margin-left: 5px;
  cursor: pointer;
}
</style>

<script>
export default {
  data() {
    return {
      daysId: [], //Dias de la semana en numero
      daysName: [],
      timeSlots:[],
      hoursId: [], //Horas del dia en numero
      recursos: [],
      reservations: {}, // Objeto para almacenar reservas
      showModal: false, // Controla si se muestra el modal
      selectedSlot: { day: "", hour: "" }, // Casilla seleccionada
      students: 0, // Número de alumnos
      currentUser: "Carlos", // Simulación del usuario logueado
      currentUserEmail: "carlosevans@example.com", // Simulación del correo del usuario logueado

    };
  },
  methods: {
    //Este metodo selecciona la combinacion de hora y dia y abre el popup con esos datos
    selectSlot(day, hour) {
      this.selectedSlot = { day, hour };
      this.students = 0;
      this.showModal = true;
    },
    //Este metodo rellena en combobox con los recursos de aulas
    async getResources() {
      try {
        const response = await fetch(
          "http://localhost:8085/bookings/previous_resources/resources"
        );
        if (!response.ok) {
          throw new Error("Error al obtener recursos de la API");
        }
        const data = await response.json();
        this.recursos = data.map((item) => item.aulaYCarritos);
      } catch (error) {
        console.error("Error al obtener recursos:", error);
        alert("Hubo un error al obtener los recursos. Intenta nuevamente.");
      }
    },
    //Este metodo obtiene los ids y las horas de los tramos de la BDD
    async getTimeRanges(){
      try {
        const response = await fetch(
          "http://localhost:8085/bookings/previous_resources/timeslots"
        );
        if (!response.ok) {
          throw new Error("Error al obtener recursos de la API");
        }
        const data = await response.json();
        this.hoursId = data.map((item) => item.id); 
        this.timeSlots = data.map((item) => item.tramosHorarios);
      } catch (error) {
        console.error("Error al obtener horas:", error);
        alert("Hubo un error al obtener las horas. Intenta nuevamente.");
      }
    },
    //Este metodo convierte la hora mostrando en la columna de las filas a su tramo correspondiente
    getTimeSlot(hourId) {
      const index = this.hoursId.indexOf(hourId);
      return index !== -1 ? this.timeSlots[index] : "";
    },
    //Este metodo confirma una reserva en base a los datos actuales y lo mete en BDD
    async confirmReservation() {
      try {
    const recursoSeleccionado = document.getElementById("roles").value;

   
    const response = await fetch('http://localhost:8085/bookings/previous_resources/bookings', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'email': this.currentUserEmail, // Header email requerido
        'recurso': recursoSeleccionado, // Header recurso requerido
        'diaDeLaSemana': this.selectedSlot.day, // Header diaDeLaSemana requerido
        'tramosHorarios': this.selectedSlot.hour, // Header tramosHorarios requerido
        'nAlumnos': this.students.toString(), // Header nAlumnos requerido (debe ser string en los headers)
      },
    });

    if (!response.ok) {

      const errorData = await response.json();
      alert('Error al reservar: ' + (errorData.message || 'Error desconocido.'));
      return false;
    }

    if (!this.reservations[this.selectedSlot.day]) {
          this.reservations[this.selectedSlot.day] = {};
        }
        this.reservations[this.selectedSlot.day][this.selectedSlot.hour] = {
          name: this.currentUser,
          students: this.students,
    };
    
    alert('Reserva completada');
    this.closeModal();
    
  } catch (error) {
    console.error('Error al conectarse a la API:', error);
    alert('Hubo un error al conectarse al servidor. Intenta nuevamente más tarde.');
    return false;
  }
      this.closeModal();
    },
    //Este metodo cierra el popup
  closeModal() {
      this.showModal = false;
    },
  },
  //El metodo mounted ejecuta los metodos de obtencion de datos al iniciar el despliegue
  mounted() {
    // Llama a la API cuando el componente se monte
    this.getResources();
    this.getTimeRanges()
    this.getDaysOfTheWeek()
  },
};
</script>

<template>
  <div>
    <label for="roles" >Aula o carrico que quieres reservar </label>
    <select id="roles"  v-model="selectedResource" @change="getReservationsByResource" aria-expanded="false">
      <option v-for="recurso in recursos" :key="recurso">{{recurso}}</option>
    </select>
  </div>
  <div>
    <table border="1">
      <thead>
        <tr>
          <th>Hora/Día</th>
          <th v-for="dayId in daysId" :key="dayId">{{ getdaysName(dayId) }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="hourId in hoursId" :key="hourId">
          <td>{{ getTimeSlot(hourId) }}</td>
          <td
            v-for="dayId in daysId"
            :key="dayId"
            class="cell"
            @click="selectSlot(dayId, hourId)">

            <div v-if="reservations[dayId]?.[hourId]">
              {{ reservations[dayId][hourId].name }} ({{ reservations[dayId][hourId].students }})
              <button class="delete-btn" @click.stop="deleteReservation(dayId, hourId)">X</button>
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
 <!-- Nueva tarjeta: Actualizar constantes -->
 <div class="update-constants-card">
    <div class="title-container">
      <h1 class="title">Actualizar constantes</h1>
    </div>
    <div class="grid">
      <!-- Selección de constante -->
      <div class="row">
        <div class="col-12">
          <div class="form-item">
            <label for="constante-select" class="form-label">Clave de la constante:</label>
            <select
              id="constante-select"
              v-model="selectedConstante"
              @change="onConstanteChange"
              class="form-select"
            >
              <option
                v-for="constante in constantes"
                :key="constante.clave"
                :value="constante"
              >
                {{ constante.clave }}
              </option>
            </select>
          </div>
        </div>
      </div>

      <!-- Input para valor de la constante -->
      <div class="row" v-if="selectedConstante">
        <div class="col-12">
          <div class="form-item">
            <label for="constante-valor" class="form-label">Valor:</label>
            <input
              id="constante-valor"
              v-model="selectedConstante.valor"
              type="text"
              class="form-input"
            />
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-12">
          <div class="form-item">
            <label for="additional-note" class="form-label">Motivo de cierre:</label>
            <textarea
              id="additional-note"
              v-model="notaAdicional"
              class="form-textarea"
              placeholder="Escribe una nota adicional antes de actualizar (opcional)"
            ></textarea>
          </div>
        </div>
      </div>

      <!-- Botón de actualización -->
      <div class="row">
        <div class="col-12">
          <button
            class="btn btn-primary btn-block"
            @click="actualizarConstanteSeleccionada"
          >
            Actualizar
          </button>
        </div>
      </div>

      <!-- Mensaje de resultado de la actualización -->
      <div class="row" v-if="mensajeActualizacion">
        <div class="col-12">
          <p :class="['update-message', mensajeColor]">
            {{ mensajeActualizacion }}
          </p>
        </div>
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
  background-color: #a7bcd3;
  border: 1px solid #1722e9;
}
.cell {
  height: 60px;
  width: 300px;
  cursor: pointer;
}
.cell:hover {
  background-color: #e9e572;
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
.update-constants-card {
  padding: 1.5rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  background: #fff;
  max-width: 600px;
  margin: 1rem auto;
}
.title-container {
  text-align: center;
  margin-bottom: 1rem;
}
.title {
  font-size: 1.5rem;
  font-weight: bold;
}
.grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.row {
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}
.col-12 {
  flex: 0 0 100%;
}
.form-item {
  display: flex;
  flex-direction: column;
}
.form-label {
  margin-bottom: 0.5rem;
  font-weight: bold;
}
.form-select,
.form-input {
  padding: 0.5rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.btn {
  padding: 0.75rem 1rem;
  font-size: 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.btn-block {
  width: 100%;
}
.btn-primary {
  background-color: #007bff;
  color: #fff;
}
.update-message {
  text-align: center;
  font-weight: bold;
}
.update-message.success {
  color: green;
}
.update-message.danger {
  color: red;
}
</style>

<script>
//import { IonApp, IonRouterOutlet } from '@ionic/vue';
//import { obtenerConstantes, actualizarConstantes } from '@/services/constantes';
//import { IonGrid, IonRow, IonCol, IonItem, IonLabel, IonText } from '@ionic/vue';
//import { IonSelect, IonSelectOption, IonInput, IonButton, IonIcon } from '@ionic/vue';

export default {
  data() {
    return {
      daysId: [], //Dias de la semana en numero
      daysName: [],//Nombre real de los dias
      timeSlots:[],//Nombre real de los tramos horarios
      hoursId: [], //Horas del dia en numero
      recursos: [],//Recursos del combobox
      selectedResource: "", // Recurso seleccionado
      reservations: {}, // Objeto para almacenar reservas
      showModal: false, // Controla si se muestra el modal
      selectedSlot: { day: "", hour: "" }, // Casilla seleccionada
      students: 0, // Número de alumnos
      currentUser: "Carlos", // Simulación del usuario logueado
      currentUserEmail: "carlosevans@example.com", // Simulación del correo del usuario logueado

    };
  },
  methods: {
    // Función para actualizar la constante seleccionada
  /*async actualizarConstanteSeleccionada(){
  try {
    const constantesActualizadas = constantes.value.map(c =>
      c.clave === selectedConstante.value.clave ? selectedConstante.value : c
    );

    await actualizarConstantes('/printers/web/constantes', toastMessage, toastColor, isToastOpen, constantesActualizadas);
    crearToast(toastMessage, toastColor, isToastOpen, 'success', 'Constante actualizada con éxito');
    mensajeActualizacion.value = 'Constantes actualizadas con éxito';
    mensajeColor.value = 'success';
  } catch (error) {
    crearToast(toastMessage, toastColor, isToastOpen, 'danger', 'Error al actualizar la constante');
    mensajeActualizacion.value = 'Error al actualizar la constante';
    mensajeColor.value = 'danger';
    throw new Error(error.message);
  }
},*/

// Función para obtener las constantes al cargar el componente
 async cargarConstantes() {
  try {
    constantes.value = await obtenerConstantes('/printers/web/constantes');

    // Seleccionar la constante "Impresión Deshabilitada" por defecto
    const impresionDeshabilitada = constantes.value.find(c => c.clave === 'Impresion Deshabilitada');
    
    if (impresionDeshabilitada) {
      selectedConstante.value = impresionDeshabilitada;
    }
  } catch (error) {
    crearToast(toastMessage, toastColor, isToastOpen, 'danger', 'Error al obtener constantes');
    mensajeActualizacion.value = 'Error al obtener constantes';
    mensajeColor.value = 'danger';
    throw new Error(error.message);
  }
},

// Función que se llama cuando el usuario selecciona una constante
  /*async onConstanteChange(){
  if (selectedConstante.value && selectedConstante.value.valor !== undefined) {
    selectedConstante.value.valor = selectedConstante.value.valor;
  } else {
    selectedConstante.value = { valor: '' };
  }
},*/

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
        if (this.recursos.length > 0) this.selectedResource = this.recursos[0];
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

    //Este metodo obtiene los datos de los dias de la semana de la BDD
    async getdaysOfTheWeek() {
      try {
        const response = await fetch(
          "http://localhost:8085/bookings/previous_resources/days_week"
        );
        if (!response.ok) {
          throw new Error("Error al obtener los días de la semana");
        }
        const data = await response.json();
        if (data && data.length > 0) {
          this.daysId = data.map((item) => item.id);
          this.daysName = data.map((item) => item.diasDeLaSemana);
        } else {
          console.error("Datos de días de la semana vacíos o inválidos.");
        }
      } catch (error) {
        console.error("Error al obtener los días:", error);
        alert("Hubo un error al obtener los días. Intenta nuevamente.");
      }
    },

    getdaysName(dayId){
      const index = this.daysId.indexOf(dayId);
      return index !== -1 ? this.daysName[index] : "";
    },

  //Este metodo rellena las celdas con las reservas segun lo que hay en el combobox
    async getReservationsByResource() {
  try {
    const recursoSeleccionado = document.getElementById("roles").value;

    const response = await fetch(
      "http://localhost:8085/bookings/previous_resources/bookings",
      {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
          "aulaYCarritos": recursoSeleccionado,
        },
      }
    );

    if (!response.ok) throw new Error("Error al obtener reservas");

    const data = await response.json();
    //console.log("Datos recibidos:", data);

    const reservasTransformadas = {};

    data.forEach(function(reserva) {
      const dia = reserva.diaSemana;
      const tramo = reserva.tramoHorario;
      const nombre = reserva.nombreYapellidos;
      const alumnos = reserva.nalumnos;
      //console.log("Procesando reserva:", dia, tramo, nombre, alumnos);

      if (!reservasTransformadas[dia]) {
        reservasTransformadas[dia] = {};
      }
      if(nombre != null){
      reservasTransformadas[dia][tramo] = {
        name: nombre || "Nulo",
        students: alumnos || 0,    
                            
      };
    }
    });

    //console.log("Reservas transformadas:", reservasTransformadas);
    this.reservations = reservasTransformadas;

  } catch (error) {
    console.error("Error al obtener reservas:", error);
    alert("Hubo un error al obtener las reservas. Intenta nuevamente.");
  }
},
    
    //Este metodo confirma una reserva en base a los datos actuales y lo mete en BDD
    async confirmReservation() {
      try {
    const recursoSeleccionado = document.getElementById("roles").value;

   
    const response = await fetch('http://localhost:8085/bookings/previous_resources/bookings', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'email': this.currentUserEmail,
        'recurso': recursoSeleccionado,
        'diaDeLaSemana': this.selectedSlot.day,
        'tramosHorarios': this.selectedSlot.hour,
        'nAlumnos': this.students.toString(),
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

    //Este metodo borra una reserva de la base de datos
    async deleteReservationFromDB(day, hour){
      try{
        const recursoSeleccionado = document.getElementById("roles").value;

        const response = await fetch('http://localhost:8085/bookings/previous_resources/bookings', {
      method: 'DELETE',
      headers: {
        'Content-Type': 'application/json',
        'email': this.currentUserEmail, 
        'recurso': recursoSeleccionado, 
        'diaDeLaSemana': day,
        'tramoHorario': hour,
      },
    });

    if (!response.ok) {

      const errorData = await response.json();
      alert('Error al borrar: ' + (errorData.message || 'Error desconocido.'));
      return false;
    }
    alert('Reserva borrrada');
      }
      catch (error) {
    console.error('Error al conectarse a la API:', error);
    alert('Hubo un error al conectarse al servidor. Intenta nuevamente más tarde.');
    return false;
  }
    },
  // Este metodo elimina la reserva de la celda seleccionada
    async deleteReservation(day, hour) {
      
      if (this.reservations[day]?.[hour]) {
    delete this.reservations[day][hour];

    if (Object.keys(this.reservations[day]).length === 0) {
      delete this.reservations[day];
    }
    await this.deleteReservationFromDB(day, hour);
  }
    },
    //Este metodo cierra el popup
  closeModal() {
      this.showModal = false;
    },
  },
  //El metodo mounted ejecuta los metodos de obtencion de datos al iniciar el despliegue
  async mounted() {
    await this.getResources();
    await this.getTimeRanges();
    await this.getdaysOfTheWeek();
    await this.getReservationsByResource();
  },
};
</script>

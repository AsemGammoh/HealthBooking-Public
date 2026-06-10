<template>
  <div>
    <nav class="navbar navbar-light bg-white shadow-sm mb-4">
      <div class="container d-flex justify-content-between align-items-center">
        <span class="fw-bold fs-5">Doctor Appointments</span>
        <router-link to="/" class="btn btn-outline-primary">
          ⇆ Switch Page
        </router-link>
      </div>
    </nav>

    <div class="container">
      <div class="card shadow-sm">
        <div class="card-header">
          <h5 class="mb-0">Appointments</h5>
        </div>

        <div class="card-body p-0">
          <table class="table table-bordered table-striped mb-0">
            <thead class="table-light">
              <tr>
                <th>Name</th>
                <th>Symptoms</th>
                <th>Time</th>
                <th>Status</th>
                <th>Update</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="appointment in appointments" :key="appointment.appointmentId">
                <td>{{ appointment.patientName }}</td>
                <td>{{ appointment.symptoms }}</td>
                <td>{{ appointment.slot }}</td>
                <td>{{ appointment.status }}</td>
                <td>
                  <select
                    class="form-select"
                    :value="appointment.status"
                    @change="event => updateStatus(appointment, event.target.value)"
                  >
                    <option>Pending</option>
                    <option>In Progress</option>
                    <option>Completed</option>
                  </select>
                </td>
              </tr>

              <tr v-if="appointments.length === 0">
                <td colspan="5" class="text-center p-3">
                  No appointments found.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div v-if="errorMessage" class="alert alert-danger mt-3">
        {{ errorMessage }}
      </div>
    </div>
  </div>
</template>

<script>
const API_BASE_URL = "https://2r4l7cheo4.execute-api.eu-north-1.amazonaws.com/prod";

export default {
  name: "AppointmentsList",

  data() {
    return {
      appointments: [],
      errorMessage: ""
    };
  },

  mounted() {
    this.fetchAppointments();
  },

  methods: {
    parseApiResponse(data) {
      if (Array.isArray(data)) {
        return data;
      }

      if (typeof data === "string") {
        return JSON.parse(data);
      }

      if (data && typeof data.body === "string") {
        return JSON.parse(data.body);
      }

      return [];
    },

    async fetchAppointments() {
      try {
        this.errorMessage = "";

        const response = await fetch(`${API_BASE_URL}/appointments`);
        const data = await response.json();

        console.log("Appointments API response:", data);

        this.appointments = this.parseApiResponse(data);
      } catch (error) {
        console.error("Failed to load appointments:", error);
        this.errorMessage = "Failed to load appointments.";
      }
    },

    async updateStatus(appointment, newStatus) {
      const url = `${API_BASE_URL}/appointments/${appointment.appointmentId}`;

      const payload = {
        status: newStatus
      };

      try {
        this.errorMessage = "";

        const response = await fetch(url, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(payload)
        });

        const result = await response.json();

        if (!response.ok) {
          throw new Error(result.message || result.error || "Failed to update status.");
        }

        appointment.status = newStatus;
        alert("Status updated!");
      } catch (error) {
        console.error("Failed to update status:", error);
        this.errorMessage = error.message || "Update failed. See console for details.";
      }
    }
  }
};
</script>

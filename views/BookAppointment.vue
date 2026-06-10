<template>
  <div>
    <nav class="navbar navbar-light bg-white shadow-sm mb-4">
      <div class="container d-flex justify-content-between align-items-center">
        <span class="fw-bold fs-5">Doctor Appointments</span>
        <router-link to="/appointments" class="btn btn-outline-primary">
          ⇆ Switch Page
        </router-link>
      </div>
    </nav>

    <div class="d-flex justify-content-center align-items-center" style="min-height: 80vh;">
      <div class="card shadow p-5" style="width: 100%; max-width: 700px;">
        <h2 class="text-center mb-4 text-primary">Book an Appointment</h2>

        <div v-if="errorMessage" class="alert alert-danger">
          {{ errorMessage }}
        </div>

        <form class="row g-3" @submit.prevent="submitAppointment">
          <div class="col-12">
            <input
              v-model="name"
              type="text"
              class="form-control"
              placeholder="Your Name"
              required
            />
          </div>

          <div class="col-12">
            <input
              v-model="symptoms"
              type="text"
              class="form-control"
              placeholder="Symptoms"
              required
            />
          </div>

          <div class="col-12">
            <select v-model="selectedSlot" class="form-select" required>
              <option disabled value="">Select a Time Slot</option>
              <option v-for="slot in slots" :key="slot" :value="slot">
                {{ slot }}
              </option>
            </select>
          </div>

          <div class="col-12">
            <button type="submit" class="btn btn-primary w-100">
              Book
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
const API_BASE_URL = "https://2r4l7cheo4.execute-api.eu-north-1.amazonaws.com/prod";

export default {
  name: "BookAppointment",

  data() {
    return {
      name: "",
      symptoms: "",
      selectedSlot: "",
      slots: [],
      errorMessage: ""
    };
  },

  mounted() {
    this.fetchSlots();
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

    getSlotValue(item) {
      if (typeof item.slot === "string") {
        return item.slot;
      }

      if (item.slot && item.slot.S) {
        return item.slot.S;
      }

      return "";
    },

    getBookedValue(item) {
      if (typeof item.isBooked === "boolean") {
        return item.isBooked;
      }

      if (item.isBooked && typeof item.isBooked.BOOL === "boolean") {
        return item.isBooked.BOOL;
      }

      return false;
    },

    async fetchSlots() {
      try {
        this.errorMessage = "";

        const response = await fetch(`${API_BASE_URL}/slots`);
        const data = await response.json();

        console.log("Slots API response:", data);

        const parsedSlots = this.parseApiResponse(data);

        this.slots = parsedSlots
          .filter(item => this.getBookedValue(item) === false)
          .map(item => this.getSlotValue(item))
          .filter(slot => slot !== "");

        console.log("Available slots:", this.slots);
      } catch (error) {
        console.error("Error loading slots:", error);
        this.errorMessage = "Failed to load available slots.";
      }
    },

    async submitAppointment() {
      const payload = {
        patientName: this.name,
        symptoms: this.symptoms,
        slot: this.selectedSlot
      };

      try {
        this.errorMessage = "";

        const response = await fetch(`${API_BASE_URL}/appointments`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(payload)
        });

        const result = await response.json();

        if (!response.ok) {
          throw new Error(result.message || result.error || "Failed to book appointment.");
        }

        alert("Appointment booked!");

        this.name = "";
        this.symptoms = "";
        this.selectedSlot = "";

        await this.fetchSlots();
      } catch (error) {
        console.error("Error booking appointment:", error);
        this.errorMessage = error.message || "Failed to book appointment.";
      }
    }
  }
};
</script>

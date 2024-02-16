<script setup>
import { computed, onMounted, ref } from "vue";
import Swal from "sweetalert2";

const appointments = ref([]);
const selectedStatus = ref();
const appointmentStatus = ref([]);
const getAppointmentStatus = () => {
  axios.get("/api/appointment-status").then((res) => {
    appointmentStatus.value = res.data;
  });
};
const getAppointments = (status) => {
  selectedStatus.value = status;
  const params = {};
  if (status) {
    params.status = status;
  }
  axios.get(`/api/appointments`, { params }).then((res) => {
    appointments.value = res.data;
  });
};

const appointmentsCount = computed(() => {
  return appointmentStatus.value
    .map((status) => status.count)
    .reduce((acc, value) => acc + value, 0);
});

const deleteAppointment = (id) => {
  Swal.fire({
    title: "Are you sure?",
    text: "You won't be able to revert this!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#3085d6",
    cancelButtonColor: "#d33",
    confirmButtonText: "Yes, delete it!",
  }).then((result) => {
    if (result.isConfirmed) {
      axios.delete(`/api/appointments/${id}`).then((res) => {
        appointments.value.data = appointments.value.data.filter(
          (appointment) => appointment.id !== id
        );
        Swal.fire({
          title: "Deleted!",
          text: "Your file has been deleted.",
          icon: "success",
        });
      });
    }
  });
};

onMounted(() => {
  getAppointments();
  getAppointmentStatus();
});
</script>
<template>
  <div class="content-header">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-6">
          <h1 class="m-0">Appointments</h1>
        </div>
        <div class="col-sm-6">
          <ol class="breadcrumb float-sm-right">
            <li class="breadcrumb-item"><a href="#">Home</a></li>
            <li class="breadcrumb-item active">Appointments</li>
          </ol>
        </div>
      </div>
    </div>
  </div>
  <div class="content">
    <div class="container-fluid">
      <div class="row">
        <div class="col-lg-12">
          <div class="d-flex justify-content-between mb-2">
            <div>
              <router-link to="/admin/appointments/create">
                <button class="btn btn-primary">
                  <i class="fa fa-plus-circle mr-1"></i> Add New Appointment
                </button>
              </router-link>
            </div>
            <div class="btn-group">
              <button
                type="button"
                class="btn"
                :class="[
                  selectedStatus === undefined
                    ? 'btn-secondary'
                    : 'btn-default',
                ]"
                @click="getAppointments()"
              >
                <span class="mr-1">All</span>
                <span class="badge badge-pill badge-info">{{
                  appointmentsCount
                }}</span>
              </button>

              <button
                type="button"
                v-for="status in appointmentStatus"
                :key="status.name"
                @click="getAppointments(status.value)"
                class="btn"
                :class="[
                  selectedStatus === status.value
                    ? 'btn-secondary'
                    : 'btn-default',
                ]"
              >
                <span class="mr-1">{{ status.name }}</span>
                <span
                  class="badge badge-pill"
                  :class="`badge-${status.color}`"
                  >{{ status.count }}</span
                >
              </button>
            </div>
          </div>
          <div class="card">
            <div class="card-body">
              <table class="table table-bordered">
                <thead>
                  <tr>
                    <th scope="col">#</th>
                    <th scope="col">Client Name</th>
                    <th scope="col">Date</th>
                    <th scope="col">Time</th>
                    <th scope="col">Status</th>
                    <th scope="col">Options</th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="(appointment, index) in appointments.data"
                    :key="appointment.id"
                  >
                    <td>{{ index + 1 }}</td>
                    <td>
                      {{ appointment.client.first_name }}
                      {{ appointment.client.last_name }}
                    </td>
                    <td>{{ appointment.start_time }}</td>
                    <td>{{ appointment.end_time }}</td>
                    <td>
                      <span
                        class="badge"
                        :class="`badge-${appointment.status.color}`"
                        >{{ appointment.status.name }}</span
                      >
                    </td>
                    <td>
                      <router-link
                        :to="`/admin/appointments/${appointment.id}/edit`"
                      >
                        <i class="fa fa-edit mr-2"></i>
                      </router-link>

                      <a
                        href="#"
                        @click.prevent="
                          ($event) => deleteAppointment(appointment.id)
                        "
                      >
                        <i class="fa fa-trash text-danger"></i>
                      </a>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

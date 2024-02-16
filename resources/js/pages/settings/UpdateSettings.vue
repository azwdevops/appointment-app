<script setup>
import { onMounted, ref } from "vue";
import { useToaster } from "../../toaster";

const settings = ref([]);
const errors = ref();

const toaster = useToaster();

const getSettings = () => {
  axios.get("/api/settings").then((res) => {
    settings.value = res.data;
  });
};

const updateSettings = () => {
  errors.value = "";
  axios
    .post("/api/settings", settings.value)
    .then((res) => {
      toaster.success("Settings updated successfully");
    })
    .catch((err) => {
      if (err.response && err.response.status === 422) {
        errors.value = err.response.data.errors;
      }
    });
};

onMounted(() => {
  getSettings();
});
</script>

<template>
  <div class="content-header">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-6">
          <h1 class="m-0">Settings</h1>
        </div>
        <div class="col-sm-6">
          <ol class="breadcrumb float-sm-right">
            <li class="breadcrumb-item"><a href="#">Home</a></li>
            <li class="breadcrumb-item active">Settings</li>
          </ol>
        </div>
      </div>
    </div>
  </div>

  <div class="content">
    <div class="container-fluid">
      <div class="row">
        <div class="col-md-6">
          <div class="card card-primary">
            <div class="card-header">
              <h3 class="card-title">General Setting</h3>
            </div>

            <form @submit.prevent="updateSettings">
              <div class="card-body">
                <div class="form-group">
                  <label for="appName">App Display Name</label>
                  <input
                    type="text"
                    v-model="settings.app_name"
                    class="form-control"
                    id="appName"
                    placeholder="Enter app display name"
                  />
                  <span
                    class="text-danger text-sm"
                    v-if="errors && errors.app_name"
                    >{{ errors.app_name[0] }}</span
                  >
                </div>
                <div class="form-group">
                  <label for="dateFormat">Date Format</label>
                  <select class="form-control" v-model="settings.date_format">
                    <option value="m/d/Y">MM/DD/YYYY</option>
                    <option value="d/m/Y">DD/MM/YYYY</option>
                    <option value="Y-m-d">YYYY-MM-DD</option>
                    <option value="F j, Y">Month DD, YYYY</option>
                    <option value="j F Y">DD Month YYYY</option>
                  </select>
                  <span
                    class="text-danger text-sm"
                    v-if="errors && errors.date_format"
                    >{{ errors.date_format[0] }}</span
                  >
                </div>
                <div class="form-group">
                  <label for="paginationLimit">Pagination Limit</label>
                  <input
                    type="text"
                    v-model="settings.pagination_limit"
                    class="form-control"
                    id="paginationLimit"
                    placeholder="Enter pagination limit"
                  />
                  <span
                    class="text-danger text-sm"
                    v-if="errors && errors.pagination_limit"
                    >{{ errors.pagination_limit[0] }}</span
                  >
                </div>
              </div>

              <div class="card-footer">
                <button type="submit" class="btn btn-primary">
                  <i class="fa fa-save mr-1"></i>Save Changes
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

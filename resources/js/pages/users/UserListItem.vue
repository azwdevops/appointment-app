<script setup>
import { ref } from "vue";
import axios from "axios";
import { formatDate } from "../../helper.js";
import { useToaster } from "../../toaster.js";
const toaster = useToaster();

const props = defineProps({
  user: Object,
  index: Number,
  selectAll: Boolean,
});

const emit = defineEmits(["userDeleted", "editUser", "confirmUserDeletion"]);

const roles = ref([
  { name: "ADMIN", value: 1 },
  { name: "USER", value: 2 },
]);

const changeRole = (user, role) => {
  axios.patch(`/api/users/${user.id}/change-role`, { role }).then((res) => {
    toaster.success("Role changed successfully");
  });
};

const toggleSelection = () => {
  emit("toggleSelection", props.user);
};
</script>


<template>
  <tr>
    <td>
      <input
        type="checkbox"
        name=""
        @change="toggleSelection"
        :checked="selectAll"
        id=""
      />
    </td>
    <td>{{ index + 1 }}</td>
    <td>{{ user.name }}</td>
    <td>{{ user.email }}</td>
    <td>{{ user.formatted_created_at }}</td>
    <td>
      <select
        name=""
        class="form-control"
        @change="changeRole(user, $event.target.value)"
      >
        <option
          v-for="role in roles"
          :key="role.value"
          :value="role.value"
          :selected="user.role === role.name"
        >
          {{ role.name }}
        </option>
      </select>
    </td>
    <td>
      <a href="#" @click.prevent="$emit('editUser', user)"
        ><i class="fa fa-edit"></i
      ></a>
      <a href="#" @click.prevent="$emit('confirmUserDeletion', user.id)"
        ><i class="fa fa-trash text-danger ml-2"></i
      ></a>
    </td>
  </tr>
</template>

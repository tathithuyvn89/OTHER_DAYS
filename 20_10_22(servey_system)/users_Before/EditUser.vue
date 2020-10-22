<template>
  <div>
    <h1>Edit User</h1>
    <UserForm :isEdit="true" :title="Edit" />
  </div>
</template>

<script>
import UserForm from './components/UserForm';

export default {
  components: { UserForm },
};
</script>

<style scoped>
h1 {
  text-align: center;
}
</style>

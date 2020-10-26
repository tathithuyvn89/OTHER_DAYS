<template>
  <div class="details-container">
    <h1>User Details</h1>
    <el-table :data="list" border style="width: 100%">
      <el-table-column prop="empoyeeCode" :label="$t('profile.employeeCode')" />
      <el-table-column prop="name" :label="$t('profile.name')" />
      <el-table-column prop="office" :label="$t('profile.office')" />
      <el-table-column prop="gender" :label="$t('profile.gender')" />
    </el-table>
    <div class="details-button">
      <el-button type="primary" @click="goToUpdatePage()"> Update </el-button>
      <el-button type="danger" @click="deleteUser()"> Delete </el-button>
    </div>
  </div>
</template>

<script>
import UserResource from '@/api/user';
const userResource = new UserResource();

export default {
  name: 'UserDetails',
  data() {
    return {
      list: [],
    };
  },
  created() {
    const id = this.$route.params && this.$route.params.id;
    this.fetchData(id);
  },
  methods: {
    fetchData(id) {
      userResource
        .get(id)
        .then((res) => {
          this.list.push(res.data);
          console.log(this.list);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    goToUpdatePage() {
      const id = this.$route.params && this.$route.params.id;
      this.$router.push({
        path: `/users/edit/${id}`,
      });
    },
    deleteUser() {
      const id = this.$route.params && this.$route.params.id;
      console.log(this.list[0].name);
      this.$confirm(
        'This is permanently delete user ' + this.list[0].name + '. Continue?',
        'Warning',
        {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning',
        }
      ).then(() => {
        userResource
          .destroy(this.list[0].id)
          .then(() => {
            this.$message({
              type: 'success',
              message: 'Delete completed',
            });
            this.$router.push({
              path: `/users`,
              // meta: { reload: true },
            });
          })

          .catch(() => {
            this.message({
              type: 'info',
              message: 'Delete canceled',
            });
          });
      });
    },
  },
};
</script>

<style scoped>
.el-table {
  border-width: 2px;
}
.details-container {
  margin: 120px 90px;
}
.details-button {
  float: right;
  margin-top: 10px;
}
</style>

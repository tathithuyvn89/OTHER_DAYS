<template>
  <div class="createPost-container">
    <el-form
      ref="postForm"
      :rules="rules"
      :model="postForm"
      label-position="center"
      label-width="200px"
      style="max-width: 500px"
    >
      <el-form-item>
        <h2>
          {{ title }}
        </h2>
      </el-form-item>
      <el-form-item :label="$t('userFormCreate.office')" prop="office">
        <el-select
          v-model="postForm.office"
          class="filter-item"
          :placeholder="$t('list.placeHolderOffice')"
        >
          <el-option label="Zone one" value="shanghai" />
          <el-option label="Zone two" value="beijing" />
          <el-option />
        </el-select>
      </el-form-item>
      <el-form-item
        :label="$t('userFormCreate.employeeCode')"
        prop="employeeCode"
      >
        <el-input v-model="postForm.employeeCode" />
      </el-form-item>
      <el-form-item :label="$t('userFormCreate.name')" prop="name">
        <el-input v-model="postForm.name" />
      </el-form-item>
      <el-form-item :label="$t('userFormCreate.gender')" prop="gender">
        <el-radio-group v-model="postForm.gender">
          <el-radio :label="$t('userFormCreate.genderType.female')">{{
            $t('userFormCreate.genderType.male')
          }}</el-radio>
          <el-radio :label="$t('userFormCreate.genderType.male')">
            {{ $t('userFormCreate.genderType.female') }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item>
        <el-button v-loading="loading" type="primary" @click="submitForm()">
          Update
        </el-button>
        <el-button @click="cancel"> Cancel </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import UserResource from '@/api/user';
const userResource = new UserResource();
const defaultForm = {
  employeeCode: '',
  name: '',
  gender: '',
  office: '',
};

export default {
  props: {
    isEdit: {
      type: Boolean,
      default: false,
    },
    title: {
      type: String,
    },
  },
  data() {
    return {
      postForm: {},
      rules: {
        employeeCode: [
          {
            required: true,
            message: this.$t('list.messEmployeeCodeRequired'),
            trigger: 'change',
          },
        ],
        name: [
          {
            required: true,
            message: this.$t('list.messNameRequired'),
            trigger: 'blur',
          },
        ],
        office: [
          {
            required: true,
            message: this.$t('list.messOfficeRequired'),
            trigger: 'blur',
          },
        ],
        gender: [
          {
            required: true,
            message: this.$t('list.messGenderRequired'),
            trigger: 'blur',
          },
        ],
      },
      loading: false,
      userCreating: false,
    };
  },
  created() {
    const id = this.$route.params && this.$route.params.id;
    if (this.isEdit) {
      this.fetchData(id);
    } else {
      this.postForm = Object.assign({}, defaultForm);
    }
  },
  methods: {
    fetchData(id) {
      userResource
        .get(id)
        .then((resp) => {
          this.postForm = resp.data;
          console.log('This is PostForm:  ', this.postForm);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    cancel() {
      this.$router.push({
        path: `/users`,
      });
      window.location.reload(forceReload);
    },
    submitForm() {
      // create new  user
      console.log('The user moi', this.postForm);
      this.$refs['postForm'].validate((valid) => {
        if (valid) {
          this.userCreating = true;
          this.loading = true;
          userResource
            .store(this.postForm)
            .then((response) => {
              this.$notify({
                message: 'Create User Success',

                type: 'success',
                duration: 5 * 1000,
              });
            })
            .catch((error) => {
              console.log(error);
            })
            .finally(() => {
              this.userCreating = false;
            });
        } else {
          return false;
        }
      });
    },
  },
};
</script>

<style scoped>
.el-form {
  border: 1px solid #ccc;
  padding-top: 20px;
  max-width: 500px;
  padding-left: 1px spl;
  padding-right: 60px;
  margin: 100px auto auto auto;
  margin-top: 119px;
}
</style>

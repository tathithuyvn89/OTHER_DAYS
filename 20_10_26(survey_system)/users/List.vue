<template>
  <div class="app-container">
    <!-- top tags -->
    <div class="filter-container">
      <el-input
        v-model="query.keyword"
        :placeholder="$t('table.keyword')"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />

      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >
        {{ $t('table.search') }}
      </el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="primary"
        icon="el-icon-plus"
        @click="handleCreate"
      >
        {{ $t('table.add') }}
      </el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px"
        type="primary"
        icon="el-icon-upload2"
        @click="handleUpload"
      >
        Import From Excel
      </el-button>
    </div>
    <!-- ======================================USER TABLE======================================================================================== -->
    <!-- Table of users -->
    <el-table
      v-loading="loading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 70%"
    >
      <el-table-column align="center" :label="$t('profile.employeeCode')">
        <template slot-scope="scope">
          <span>{{ scope.row.employeeCode }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" :label="$t('profile.name')">
        <template slot-scope="scope">
          <!-- <router-link :to="'/users/details/' + scope.row.id"> -->
          <span class="user-links" @click="userDetails(scope.row.id)"
            >{{ scope.row.name }} + {{ scope.row.id }}</span
          >
          <!-- </router-link> -->
        </template>
      </el-table-column>

      <el-table-column align="center" :label="$t('profile.office')">
        <template slot-scope="scope">
          <span>{{ scope.row.office }}</span>
        </template>
      </el-table-column>

      <el-table-column align="center" :label="$t('profile.gender')" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.gender }}</span>
        </template>
      </el-table-column>
    </el-table>
    <!-- =============================================PAGINATION============================================================================== -->
    <!-- Pagination  -->
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="query.page"
      :limit.sync="query.limit"
      @pagination="getList"
    />
    <!-- ==========================================DETAILS USER========================================================================= -->
    <!-- pop-up details user  -->
    <el-dialog
      title="User Information"
      :visible.sync="dialogDetailsVisible"
      width="900px"
      @close="closeDialog('ruleEditForm')"
    >
      <div class="details-container">
        <el-table :data="listDetails" border style="width: 100%">
          <el-table-column
            prop="empoyeeCode"
            :label="$t('profile.employeeCode')"
          />
          <el-table-column prop="name" :label="$t('profile.name')" />
          <el-table-column prop="office" :label="$t('profile.office')" />
          <el-table-column prop="gender" :label="$t('profile.gender')" />
        </el-table>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="goToUpdatePage()">
            Update
          </el-button>
          <el-button type="danger" @click="deleteUser()"> Delete </el-button>
        </div>
      </div>
    </el-dialog>
    <!-- =============================================USER EDIT================================================================ -->
    <!-- pop-up  Update user  -->
    <el-dialog
      width="900px"
      title="User Edit"
      :visible.sync="dialogEditVisible"
    >
      <div v-loading="userCreating" class="form-container">
        <el-form
          ref="userForm"
          :rules="rules"
          :model="newUser"
          label-position="left"
          label-width="120px"
          style="max-width: 500px"
        >
          <el-form-item :label="$t('userFormCreate.office')" prop="office">
            <el-select
              v-model="newUser.office"
              class="filter-item"
              :placeholder="$t('list.placeHolderOffice')"
            >
              <el-option
                v-for="item in offices"
                :key="item"
                :label="item.office_name | uppercaseFirst"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item
            :label="$t('userFormCreate.employeeCode')"
            prop="employeeCode"
          >
            <el-input v-model="newUser.employeeCode" />
          </el-form-item>
          <el-form-item :label="$t('userFormCreate.name')" prop="name">
            <el-input v-model="newUser.name" />
          </el-form-item>
          <el-form-item :label="$t('userFormCreate.gender')" prop="gender">
            <el-radio-group v-model="newUser.gender">
              <el-radio :label="$t('userFormCreate.genderType.female')">{{
                $t('userFormCreate.genderType.male')
              }}</el-radio>
              <el-radio :label="$t('userFormCreate.genderType.male')">
                {{ $t('userFormCreate.genderType.female') }}
              </el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogEditVisible = false">
            {{ $t('profile.cancel') }}
          </el-button>
          <el-button type="primary" @click="createUser()">
            {{ $t('profile.confirm') }}
          </el-button>
        </div>
      </div>
    </el-dialog>
    <!-- =============================================USER CREATE================================================================ -->
    <!-- pop-up Create new user  -->
    <el-dialog
      width="700px"
      :title="$t('userFormCreate.titleCreate')"
      :visible.sync="dialogFormVisible"
    >
      <div v-loading="userCreating" class="form-container">
        <el-form
          ref="userForm"
          :rules="rules"
          :model="newUser"
          label-position="left"
          label-width="120px"
          style="max-width: 500px"
        >
          <el-form-item :label="$t('userFormCreate.office')" prop="office">
            <el-select
              v-model="newUser.office"
              class="filter-item"
              :placeholder="$t('list.placeHolderOffice')"
            >
              <el-option
                v-for="item in offices"
                :key="item"
                :label="item.office_name | uppercaseFirst"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item
            :label="$t('userFormCreate.employeeCode')"
            prop="employeeCode"
          >
            <el-input v-model="newUser.employeeCode" />
          </el-form-item>
          <el-form-item :label="$t('userFormCreate.name')" prop="name">
            <el-input v-model="newUser.name" />
          </el-form-item>
          <el-form-item :label="$t('userFormCreate.gender')" prop="gender">
            <el-radio-group v-model="newUser.gender">
              <el-radio :label="$t('userFormCreate.genderType.female')">{{
                $t('userFormCreate.genderType.male')
              }}</el-radio>
              <el-radio :label="$t('userFormCreate.genderType.male')">
                {{ $t('userFormCreate.genderType.female') }}
              </el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">
            {{ $t('profile.cancel') }}
          </el-button>
          <el-button type="primary" @click="createUser()">
            {{ $t('profile.confirm') }}
          </el-button>
        </div>
      </div>
    </el-dialog>
    <!-- =============================================IMPORT FILE AT HERE================================================================ -->
    <!-- pop-up Create new user  -->
  </div>
</template>

<script>
import Pagination from '@/components/Pagination'; // Secondary package based on el-pagination
import UserResource from '@/api/user';
import Resource from '@/api/resource';
import waves from '@/directive/waves'; // Waves directive
import permission from '@/directive/permission'; // Permission directive
import checkPermission from '@/utils/permission'; // Permission checking
import { fetchList } from '@/api/office';
const userResource = new UserResource();

export default {
  name: 'UserList',
  components: { Pagination },
  directives: { waves, permission },
  data() {
    var validateConfirmPassword = (rule, value, callback) => {
      if (value !== this.newUser.password) {
        callback(new Error(this.$t('profile.messPassMismatched')));
      } else {
        callback();
      }
    };
    return {
      list: null,
      total: 0,
      loading: true,
      downloading: false,
      userCreating: false,
      listDetails: [],
      query: {
        page: 1,
        limit: 15,
        keyword: '',
        roles: '',
      },
      selectedUserId: '', // [User Details]=>use

      offices: [], //[User Create + Edit] =>use
      newUser: {}, //[User create] =>user
      dialogFormVisible: false, //[User create] =>use
      dialogDetailsVisible: false, //[User Details] =>use
      dialogEditVisible: false, //[User Edit] =>use
      currentUserId: 0,
      currentUser: {
        name: '',
        permissions: [],
        rolePermissions: [],
      },

      //create rules for form user
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

      isproces: false,
      activeActivity: 'first',
      updating: false,

      value: 'user',
      last_page: undefined,
      listQuery: undefined,
      lengthData: undefined,
      permissionProps: {
        children: 'children',
        label: 'name',
        disabled: 'disabled',
      },
      permissions: [],
    };
  },
  computed: {
    // Popup Edit user
    getdata() {
      console.log('GET DATA');
      this.getList();
    },
    normalizedMenuPermissions() {
      let tmp = [];
      this.currentUser.permissions.role.forEach((permission) => {
        tmp.push({
          id: permission.id,
          name: permission.name,
          disabled: true,
        });
      });
      const rolePermissions = {
        id: -1, // Just a faked ID
        name: 'Inherited from role',
        disabled: true,
        children: this.classifyPermissions(tmp).menu,
      };
      tmp = this.menuPermissions.filter(
        (permission) =>
          !this.currentUser.permissions.role.find((p) => p.id === permission.id)
      );
      const userPermissions = {
        id: 0, // Faked ID
        name: 'Extra menus',
        children: tmp,
        disabled: tmp.length === 0,
      };
      return [rolePermissions, userPermissions];
    },
    normalizedOtherPermissions() {
      let tmp = [];
      this.currentUser.permissions.role.forEach((permission) => {
        tmp.push({
          id: permission.id,
          name: permission.name,
          disabled: true,
        });
      });
      const rolePermissions = {
        id: -1,
        name: 'Inherited from role',
        disabled: true,
        children: this.classifyPermissions(tmp).other,
      };
      tmp = this.otherPermissions.filter(
        (permission) =>
          !this.currentUser.permissions.role.find((p) => p.id === permission.id)
      );
      const userPermissions = {
        id: 0,
        name: 'Extra permissions',
        children: tmp,
        disabled: tmp.length === 0,
      };
      return [rolePermissions, userPermissions];
    },
    userMenuPermissions() {
      return this.classifyPermissions(this.userPermissions).menu;
    },
    userOtherPermissions() {
      return this.classifyPermissions(this.userPermissions).other;
    },
    userPermissions() {
      return this.currentUser.permissions.role.concat(
        this.currentUser.permissions.user
      );
    },
  },
  mounted() {
    console.log('MOUNTED LIST');
  },
  //==========================================CREATED()===============================================================
  created() {
    // initialization
    this.resetNewUser();
    // call list user from api
    this.getList();

    if (checkPermission(['manage permission'])) {
      this.getPermissions();
    }
    this.getOfficeList();
  },
  // ======================================METHODS==============================================================
  methods: {
    async getOfficeList() {
      const response = await fetchList(this.listQuery);

      this.offices = [];
      for (var i = 1; i <= response.last_page; i++) {
        const pageIndex = { page: i };
        const response = await fetchList(pageIndex);
        this.offices = this.allOffices.concat(response.data);
      }
    },
    // METHOD use  user detail popup
    userDetails(id) {
      this.dialogDetailsVisible = true;
      this.selectedUserId = id;
      this.listDetails = [];
      userResource
        .get(this.selectedUserId)
        .then((response) => {
          const selectedUser = response.data;
          console.log('user is: ', selectedUser);
          this.listDetails.push(selectedUser);
        })
        .catch((error) => console.log(error));
      console.log('this is id da duoc select', this.currentUserId);
    },

    deleteUser() {
      this.$confirm(
        'This is permanently delete user ' +
          this.listDetails[0].name +
          '. Continue?',
        'Warning',
        {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning',
        }
      ).then(() => {
        userResource
          .destroy(this.listDetails[0].id)
          .then(() => {
            this.$message({
              type: 'success',
              message: 'Delete completed',
            });
            this.dialogDetailsVisible = false;
            this.handleFilter();
          })
          .catch(() => {
            this.message({
              type: 'info',
              message: 'Delete canceled',
            });
          })
          .finally(() => {
            this.dialogDetailsVisible = false;
          });
      });
    },
    //===================================EDIT page================================================
    goToUpdatePage() {
      this.dialogDetailsVisible = false;

      this.dialogEditVisible = true;
      this.newUser = this.listDetails[0];
    },
    submitForm() {
      // create new  user
      console.log('The user moi', this.newUser);
      this.$refs['userForm'].validate((valid) => {
        if (valid) {
          this.userCreating = true;
          this.loading = true;
          userResource

            .store(this.newUser)
            .then((response) => {
              console.log('this is response when edit user info: ', response);
              this.$notify({
                message: 'Edit user Success',

                type: 'success',
                duration: 5 * 1000,
              });
              this.handleFilter();
              this.resetNewUser();
            })
            .catch((error) => {
              console.log(error);
            })
            .finally(() => {
              this.userCreating = false;
              this.dialogEditVisible = false;
            });
        } else {
          return false;
        }
      });
    },
    // take the officeList
    async getOfficeList() {
      const { limit, page } = this.query;
      this.loading = true;
      const { data, meta } = await fetchList(this.query);
      console.log('Data from office api', data);
      this.offices = data;
      console.log('this is meta in office', meta);
      console.log('This is list office', this.offices);
    },
    handleUpload() {
      this.$router.push({ path: `/users/import` });
    },
    checkPermission,
    async getPermissions() {
      const { data } = await permissionResource.list({});
      const { all, menu, other } = this.classifyPermissions(data);
      this.permissions = all;
      this.menuPermissions = menu;
      this.otherPermissions = other;
    },
    openPop(row) {
      // open pop edit user
      this.dialogVisible = true;
      this.ruleForm = Object.assign({}, row);
      this.value = row.roles[0];
    },
    closeDialog(formName) {
      // Event popup close
      this.$refs[formName].resetFields();
      this.ruleForm.name = '';
      this.ruleForm.email = '';
      this.ruleForm.roles = ' ';
      this.value = '';
    },
    //==================================FETCH USERSLIST FROM API===========================================
    async getList() {
      // take the data
      const { limit, page } = this.query;
      this.loading = true;
      const { data, meta } = await userResource.list(this.query);
      this.list = data;
      console.log('this is list', this.list);
      this.list.forEach((element, index) => {
        element['index'] = (page - 1) * limit + index + 1;
      });
      this.total = meta.total;
      this.last_page = meta.last_page;
      this.lengthData = data.length;
      this.loading = false;
    },

    //==============================CREATE POP-UP(CREATE USER) WHEN CLICK=================================
    handleCreate() {
      this.resetNewUser();
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs['userForm'].clearValidate();
      });
    },
    //========================================FILTER DATA================================================
    handleFilter() {
      this.query.page = 1;
      this.getList();
    },
    //==========================================CREATE USER============================================
    createUser() {
      // create new  user
      console.log('The user moi', this.newUser);
      this.$refs['userForm'].validate((valid) => {
        if (valid) {
          this.userCreating = true;

          userResource
            .store(this.newUser)
            .then((response) => {
              this.$message({
                message:
                  this.$t('list.messCfNewUserFont') +
                  this.newUser.name +
                  '(' +
                  this.newUser.email +
                  this.$t('list.messCfNewUserTail'),
                type: 'success',
                duration: 5 * 1000,
              });
              this.last_page = response.page_infor.last_page;
              this.listQuery.page = response.page_infor.last_page;
              this.total = response.page_infor.total;
              this.query.page = response.page_infor.last_page;
              this.resetNewUser();
              this.dialogFormVisible = false;
              this.handleFilter();
            })
            .catch((error) => {
              console.log(error);
            })
            .finally(() => {
              this.userCreating = false;
              this.dialogFormVisible = false;
            });
        } else {
          return false;
        }
      });
    },
    //===========================================RESET FORM USER=======================================================================
    resetNewUser() {
      this.newUser = {
        name: '',
        employeeCode: '',
        gender: '',
        office: '',
      };
    },
  },
};
</script>

<style lang="scss" scoped>
.edit-input {
  padding-right: 100px;
}
.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
.dialog-footer {
  text-align: center;

  padding-top: 0;

  // margin-left: 150px;
}
.app-container {
  flex: 1;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
  .block {
    float: left;
    min-width: 250px;
  }
  .clear-left {
    clear: left;
  }
  .inputClassname {
    margin-top: 30px;
  }

  .btnForm {
    text-align: right;
  }

  .pagination {
    margin-top: 10px;
    text-align: right;
  }
  .center {
    text-align: center;
  }

  .user-links:hover {
    text-decoration: underline;
    cursor: pointer;
    color: #1251c7;
  }
}
</style>

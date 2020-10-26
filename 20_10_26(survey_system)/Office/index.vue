<template>
  <div class="app-container">
    <!-- =============================FILTER CONTAINER================================================================= -->
    <div class="filter-container">
      <el-button type="success" @click="createOffice">
        {{ $t('button.create') }}
      </el-button>
      <el-button type="danger" @click="deleteManyItems()">
        {{ $t('button.delete') }}
      </el-button>
    </div>
    <!-- ============================================TABLE OFFICES================================================================ -->
    <el-table
      :key="tableKey"
      :data="officesList"
      v-loading="loadingList"
      :default-sort="{ prop: 'office_name' }"
      border
      fit
      stripe
      highlight-current-row
      style="width: 100%"
      class="tableOfficesList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column
        :label="$t('table.col_office_name')"
        align="center"
        sortable
        prop="office_name"
        min-width="200"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.office_name }}</span>
        </template>
      </el-table-column>

      <el-table-column
        v-if="hidden != true"
        :label="$t('table.col_operations')"
        align="center"
        min-width="150"
      >
        <template slot-scope="{ row }">
          <el-button
            type="primary"
            icon="el-icon-edit"
            circle
            @click="handleEdit(row)"
          />
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            @click="handleDeleteOne(row)"
          />
        </template>
      </el-table-column>
    </el-table>
    <!-- ===============================================PAGINATION================================================================== -->
    <el-pagination
      :current-page.sync="listQuery.page"
      :page-size="10"
      :total="total"
      background
      layout="total, prev, pager, next, jumper"
      class="pagination"
      @current-change="getList"
    />
    <!-- =============================DIALOG FOR CREATE or EDIT OFFICE============================================================= -->
    <el-dialog
      :title="titleCreateOrUpdate"
      :visible.sync="dialogCreateOrEdit"
      @close="closeDialog('formOffice')"
    >
      <el-form
        ref="formOffice"
        :model="formOffice"
        :rules="rules"
        class="form-office"
        label-width="150px"
      >
        <el-form-item
          :label="$t('popup.office.office_name')"
          prop="office_name"
          class="input-className"
        >
          <el-input
            v-model="formOffice.office_name"
            :placeholder="$t('popup.office.office_name')"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogCreateOrEdit = false"> Close </el-button>
        <el-button :disabled="isProcess" @click="submit()" type="primary"
          >Save
        </el-button>
      </div>
    </el-dialog>
    <!-- =============================DIALOG FOR DELETE OFFICE============================================================= -->
    <el-dialog
      title="Delete this user"
      :visible.sync="dialogDeleteOneOrMany"
      center
    >
      <div class="delete-container">
        <h2>Do you want delete ?</h2>

        <el-table :data="deleteItems" border width="250">
          <el-table-column type="index" width="100"> </el-table-column>
          <el-table-column prop="office_name" label="Office Name" width="300">
          </el-table-column>
        </el-table>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancelDialog"> Close </el-button>
        <el-button type="danger" @click="confirmDelete()"> Delete </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { fetchList, addOffice, deleteOffice, editOffice } from '@/api/office';
import { updateStatusDoneSurvey } from '../../../api/respondent';
export default {
  data() {
    return {
      currentNumberRow: 10, //Number of office per page
      officesList: [],
      multipleSelection: [],
      dialogCreateOrEdit: false,
      dialogDeleteOneOrMany: false,
      allOffices: [],
      formOffice: {
        office_name: '',
      },
      tempActions: '', //Check the event whether the user wants to Add or Edit
      rules: {
        office_name: [
          {
            required: true,
            message: 'Name is required',
            trigger: ['blur', 'change'],
          },
        ],
      },
      tableKey: 0,
      total: 0,
      last_page: undefined,
      lengthData: undefined, //Remain data on the page
      listQuery: {
        //Query to call api
        page: 1,
        keyword: '',
      },
      loadingList: true,
      hidden: false,
      temp: undefined,
      isProcess: false,
      titleCreateOrUpdate: undefined,
      deleteItems: [],
    };
  },
  created() {
    this.getList(); //call page 1 when loaded page
  },

  methods: {
    async getList() {
      this.loadingList = true;
      const response = await fetchList(this.listQuery);
      this.officesList = response.data;
      this.total = response.total;
      this.last_page = response.last_page;
      console.log('this last_page', this.last_page);
      this.lengthData = response.data.length;
      this.allOffices = [];
      for (var i = 1; i <= this.last_page; i++) {
        const pageIndex = { page: i };
        const response = await fetchList(pageIndex);
        this.allOffices = this.allOffices.concat(response.data);
      }

      this.loadingList = false;
    },

    toggleSelection(rows) {
      if (rows) {
        rows.forEach((row) => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = [];
      for (var i = 0; i < val.length; i++) {
        this.multipleSelection.push(val[i].id);
      }
    },
    confirmDelete() {
      if (this.tempActionDelete === 'DeleteOne') {
        this.handleDelete(this.temp);
      }
      if (this.tempActionDelete === 'DeleteMany') {
        this.deleteMany();
      }
    },
    submit() {
      this.$refs['formOffice'].validate((valid) => {
        if (valid) {
          if (this.isExistOffice(this.formOffice.office_name)) {
            this.$message({
              message: 'Office name is exist',
              type: 'error',
            });
            return;
          }
          this.isProcess = true;

          if (this.tempActions === 'create') {
            const response = addOffice(this.formOffice);
            response.then((data) => {
              if (data.message === 'successfully') {
                console.log('This is data when create new office', data);
                this.isProcess = false;
                this.dialogCreateOrEdit = false;
                this.$message({
                  message: 'successfully',
                  type: 'success',
                });
                this.loadingList = true;
                this.officesList = data.page_infor.data;
                this.last_page = data.page_infor.last_page;
                this.listQuery.page = data.page_infor.last_page;
                this.total = data.page_infor.total;
                this.getList(); //Call Api
                this.loadingList = false;
              } else {
                this.isProcess = false;
                this.dialogCreateOrEdit = false;
                this.$message({
                  message: 'Create office Failed',
                  type: 'error',
                });
              }
            });
          }
          if (this.tempActions === 'edit') {
            const responseEdit = editOffice({
              id: this.formOffice.id,

              office_name: this.formOffice.office_name,
              current_page: this.listQuery.page,
            });
            responseEdit.then((data) => {
              if (data.message === 'Update Successfully') {
                this.isProcess = false;
                this.dialogCreateOrEdit = false;
                this.$message({
                  message: 'Edit success',
                  type: 'success',
                });
                this.loadingList = true;
                this.officesList = data.page_infor.data;
                (this.last_page = data.page_infor.data),
                  (this.listQuery.page = data.page_infor.current_page);
                this.total = data.page_infor.total;
                this.listQuery.keyword = '';
                this.getList(); //Call Api

                this.loadingList = false;
              } else {
                this.isProcess = false;
                this.dialogCreateOrEdit = false;
                this.$message({
                  message: 'Update Failed',
                  type: 'error',
                });
              }
            });
          }
        }
      });
    },

    async deleteMany() {
      let deleteData = Object.values(this.multipleSelection);
      console.log('Multipleselection is : ', this.multipleSelection);
      console.log('This is deleteData', deleteData);
      if (this.multipleSelection.length === 0) {
        this.$message({
          message: this.$t('popup.office.notify.notchoseDelete'),
          type: 'warning',
        });
        this.dialogDeleteOneOrMany = false;
      } else if (this.multipleSelection.length > 0) {
        var current_page = this.listQuery.page;
        this.lengthData = this.lengthData - this.multipleSelection.length;
        if (this.lengthData > 1) {
          current_page = this.listQuery.page;
        }
        if (this.lengthData === 1 || this.lengthData === 0) {
          if (this.listQuery.page === this.last_page) {
            current_page = this.listQuery.page - 1;
            this.listQuery.page -= 1;
          }
        }
        const resDeleteMany = await deleteOffice({
          list_id: deleteData,
          current_page: current_page,
        });

        if (resDeleteMany.message === 'Delete successfully') {
          this.$message({
            message: this.$t('popup.office.notify.deleteSuccess'),
            type: 'success',
          });
          this.loadingList = true;
          this.officesList = resDeleteMany.page_infor.data;
          this.total = resDeleteMany.page_infor.total;
          this.last_page = resDeleteMany.page_infor.last_page;
          this.listQuery.page = resDeleteMany.page_infor.current_page;
          this.lengthData = resDeleteMany.page_infor.data.length;
          this.listQuery.keyword = '';
          this.getList(); //Call Api

          this.loadingList = false;

          this.dialogDeleteOneOrMany = false;
        } else {
          this.$message({
            message: this.$t('popup.office.notify.deleteFailed'),
            type: 'warning',
          });
        }
      }
      this.deleteItems = [];
    },
    //Cancel Dialog Delete Office for dialogDeleteOneOrMany
    cancelDialog() {
      this.dialogDeleteOneOrMany = false;
      this.deleteItems = [];
      this.multipleSelection = [];
    },
    // create Office fordialogCreateOrEdit
    createOffice() {
      this.dialogCreateOrEdit = true;
      this.tempActions = 'create';
      this.titleCreateOrUpdate = 'Create new Office';
    },
    // edit Office fordialogCreateOrEdit
    handleEdit(row) {
      this.dialogCreateOrEdit = true;
      this.tempActions = 'edit';
      (this.titleCreateOrUpdate = 'Edit Office Information'),
        (this.formOffice = Object.assign({}, row));
    },
    deleteManyItems() {
      this.deleteItems = [];

      this.dialogDeleteOneOrMany = true;

      this.tempActionDelete = 'DeleteMany';
      let deleteData = Object.values(this.multipleSelection);
      //Test
      for (let i = 0; i < deleteData.length; i++) {
        const office = this.allOffices.filter(
          (office) => office.id === deleteData[i]
        );
        this.deleteItems = this.deleteItems.concat(office);
      }
      console.log('test', this.deleteItems);
    },

    handleDeleteOne(row) {
      this.deleteItems = [];
      this.dialogDeleteOneOrMany = true;
      this.tempActionDelete = 'DeleteOne';
      this.temp = row;
      const office = Object.assign({}, row);
      this.deleteItems.push(office);
    },

    async handleDelete(row) {
      const office = Object.assign({}, row);
      console.log('Office form delete', office);
      var current_page = this.listQuery.page;
      if (this.lengthData > 1) {
        current_page = this.listQuery.page;
      } else if (this.lengthData === 1) {
        current_page = this.listQuery.page - 1;
      }
      const response = await deleteOffice({
        list_id: [office.id],
        current_page: current_page,
      });

      if (response.message === 'Delete successfully') {
        this.$message({
          message: this.$t('popup.office.notify.deleteSuccess'),
          type: 'success',
        });
        this.loadingList = true;
        this.officesList = response.page_infor.data;
        this.total = response.page_infor.total;
        this.listQuery.last_page = response.page_infor.last_page;
        this.listQuery.page = response.page_infor.current_page;
        this.lengthData = response.page_infor.data.length;
        this.getList(); //Call Api

        this.loadingList = false;
        this.dialogDeleteOneOrMany = false;
      } else {
        this.$message({
          message: this.$t('popup.office.notify.deleteFailed'),
          type: 'warning',
        });
      }
      this.deleteItems = [];
    },

    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          return true;
        } else {
          return false;
        }
      });
    },

    closeDialog(formName) {
      this.$refs[formName].resetFields();
      this.rules.office_name = '';
    },
    isExistOffice(name) {
      const office = this.allOffices.filter(
        (office) => office.office_name === name
      );
      if (office.length > 0) {
        return true;
      }
      return false;
    },
  },
};
</script>

<style scoped>
.input-search {
  margin-left: 10px;
  width: 200px;
}
.delete-container {
  text-align: initial;
  padding: 25px 179px 30px;
}
</style>
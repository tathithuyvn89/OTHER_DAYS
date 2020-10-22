<template>
  <div class="dashboard-container">
    <!-- Import button -->
    <span>
      {{ departments }}
    </span>
    ===============================================
    <span>
      {{ listStaff }}
    </span>
    <div class="button_group">
      <a
        href="javascript:;"
        class="button_s my_file el-button button_s el-button--primary el-button--small"
      >
        <input
          id="upload"
          type="file"
          class="my_input"
          @change="importExcel"
        >Import
      </a>
    </div>
    <!-- Import button -->
    <!-- Show date from exel to table -->
    <!-- You can create other table component -->
    <div class="myTable">
      <el-table
        v-loading="tableLoading"
        :data="dataArr"
        :span-method="objectSpanMethod"
        border
      >
        <el-table-column
          v-for="(item, i) in tableColumn"
          :key="i"
          :prop="item.prop"
          :label="item.label"
          :width="item.width"
        />
      </el-table>
    </div>
  </div>
</template>

<script>
import xlsx from 'xlsx';
import axios from 'axios';
export default {
  name: 'Dashboard',
  data() {
    return {
      departments: [],
      dataArr: [],
      listStaff: [],
      listStaffAfter: [],
      tableColumn: [],
      tableLoading: false,
    };
  },
  created() {
    // this.getDepartments();
  },
  mounted() {
    axios
      .get('http://localhost:8980/api/v1/departments')
      .then((response) => (this.departments = response.data));
  },
  methods: {
    /**
     * Import table
     */
    // async getDepartments() {
    //   const data = await departmentResource.list({});
    //   console.log(data);
    //   this.departments = data;
    //   console.log(data);
    //   console.log(this.departments);
    // },
    setTable(headers, excellist) {
      const tableTitleData = [];
      const tableMapTitle = {};
      headers.forEach((_, i) => {
        tableMapTitle[_] = 'prop' + i;
        tableTitleData.push({
          prop: 'prop' + i,
          label: _,
          width: 100,
        });
      });
      // Mapping table coontent attribute name is English
      const newTableData = [];
      excellist.forEach((_) => {
        const newObj = {};
        Object.keys(_).forEach((key) => {
          newObj[tableMapTitle[key]] = _[key];
        });
        newTableData.push(newObj);
      });
      console.log('newTableData:', newTableData);
      this.tableColumn = tableTitleData;
      this.dataArr = newTableData;
    },
    getHeader(sheet) {
      const XLSX = xlsx;
      const headers = [];
      const range = XLSX.utils.decode_range(sheet['!ref']);
      let C;
      /* Get cell value start in the first row */
      const R = range.s.r;
      let i = 0;
      for (C = range.s.c; C <= range.e.c; ++C) {
        var cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })];
        var hdr = 'UNKNOWN' + C;
        if (cell && cell.t) {
          hdr = XLSX.utils.format_cell(cell);
        }
        if (hdr.indexOf('UNKNOWN') > -1) {
          if (!i) {
            hdr = '__EMPTY';
          } else {
            hdr = '__EMPTY' + i;
          }
          i++;
        }
        headers.push(hdr);
      }
      return headers;
    },
    // Import Table
    importExcel(e) {
      const files = e.target.files;
      console.log(files);
      if (!files.length) {
        return;
      } else if (!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())) {
        return alert(
          'The upload format is incorrect. Please upload xls or xlsx format'
        );
      }
      const fileReader = new FileReader();
      fileReader.onload = (ev) => {
        try {
          const data = ev.target.result;
          const XLSX = xlsx;
          const workbook = XLSX.read(data, {
            type: 'binary',
          });
          const wsname = workbook.SheetNames[1]; // Take the first sheet，wb.SheetNames[0] :Take the name of the first sheet in the sheets
          // Chuyen thanh SheetNames[0] sau khi test xong
          const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); // Generate JSON table content，wb.Sheets[Sheet名]    Get the data of the first sheet
          const excellist = []; // Clear received data
          // Edit data
          for (let i = 0; i < ws.length; i++) {
            excellist.push(ws[i]);
          }
          this.setTable(headers, excellist);
          for (let i = 0; i < excellist.length; i++) {
            const name = excellist[i].department;
            excellist[i].department = this.findOfficeIdByName(name);
          }
          this.listStaff = excellist;
          axios
            .post(
              'http://localhost:8980/api/v1/staffs/createlist',
              this.listStaff
            )
            .then((response) => {
              this.listStaffAfter = response;
            })
            .catch((error) => {
              console.log(error);
            });
          // At this point, you get an array containing objects that need to be processed
          const a = workbook.Sheets[workbook.SheetNames[1]]; // Chuyen SheetNames[0] sau khi test xong
          const headers = this.getHeader(a);

          // Chuyen excellist sang id cua deparment
        } catch (e) {
          return alert('Read failure!');
        }
      };
      fileReader.readAsBinaryString(files[0]);
      var input = document.getElementById('upload');
      input.value = '';
    },
    findOfficeIdByName(name) {
      for (let i = 0; i < this.departments.length; i++) {
        if (this.departments[i].name === name) {
          return { id: this.departments[i].id };
        }
      }
    },
  },
};
</script>

<style lang="scss" scoped>
// Button style
.button_group {
  .button_s {
    width: 78px;
    margin: 5px 10px 5px 5px;
  }
  .button_m {
    width: 100px;
    margin: 5px 10px 5px 5px;
  }
  .my_file {
    position: relative;
    .my_input {
      position: absolute;
      opacity: 0;
      width: 78px;
      height: 30px;
      top: 0;
      left: 0;
    }
  }
}
// Button style
</style>

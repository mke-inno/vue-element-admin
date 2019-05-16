<template>
  <div class="app-container">
    <upload-excel-component :on-success="handleSuccess" :before-upload="beforeUpload" />
    <el-table :data="tableData" border highlight-current-row style="width: 100%;margin-top:20px;margin-bottom:40px">
      <el-table-column v-for="item of tableHeader" :key="item" :prop="item" :label="item" />
    </el-table>
    <div>
      <el-form v-if="tableData.length > 0" ref="postForm" :rules="rules" :model="postForm" class="form-container">
        <el-form-item label="Client Id Field Name" prop="clientFieldName">
          <el-select v-model="postForm.clientFieldName" placeholder="Select">
            <el-option
              v-for="item in tableHeader"
              :key="item"
              :label="item"
              :value="item"
            />
          </el-select>
        </el-form-item>
      </el-form>
    </div>
    <div>
      <el-button v-if="tableData.length > 0" type="primary" @click="handleNext">
        Next
      </el-button>
    </div>
  </div>
</template>

<script>
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import CustomerIdField from './components/CustomerIdField'

const defaultForm = {
  clientFieldName: ''
}

export default {
  name: 'UploadExcel',
  components: { UploadExcelComponent, CustomerIdField },
  data() {
    return {
      tableData: [],
      tableHeader: [],
      postForm: Object.assign({}, defaultForm),
      rules: {
        clientFieldName: [{ required: true, message: 'Please select client id field name', trigger: 'change' }]
      }
    }
  },
  methods: {
    handleNext() {
      this.$refs.postForm.validate((valid) => {
        console.log(valid)
      })

      if (this.postForm.clientFieldName !== '') {
        this.$router.push({
          name: 'CreateCampaign',
          params: { tableData: this.tableData, tableHeader: this.tableHeader, clientFieldName: this.postForm.clientFieldName }
        })
      } else {
        this.$notify({
          title: 'Error',
          message: '[Client Name] field is required',
          type: 'error',
          duration: 2000
        })
      }
    },
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 1

      if (isLt1M) {
        return true
      }

      this.$message({
        message: 'Please do not upload files larger than 1m in size.',
        type: 'warning'
      })
      return false
    },
    handleSuccess({ results, header }) {
      this.tableData = results
      this.tableHeader = header
    }
  }
}
</script>

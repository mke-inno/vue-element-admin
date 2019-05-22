<template>
  <div class="app-container">
    <aside v-if="isDoneSubmit">
      Your data has been posted!.<br>
      You can access dashboard with this link <a v-bind:href="resultUrl" target="_blank"> {{ resultUrl }}</a>
    </aside>
    <div>
      <el-row :gutter="20">
        <el-col v-for="(item, index) in examples" :key="index" :span="6">
          <example-card :title="item.title" :content="item.content" :count="index + 1" />
        </el-col>
      </el-row>
      <br>
      <el-row>
        <el-button v-if="examples.length > 0" :disabled="isDoneSubmit" type="primary" @click="handleSubmit">
          Submit
        </el-button>
      </el-row>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import ExampleCard from './components/ExampleCard'

const axios = require('axios')

function insert(str, index, value) {
  return str.substr(0, index) + value + str.substr(index)
}

function escapeRegExp(str) {
  return str.replace(/([.*+?^=!:${}()|\[\]\/\\])/g, '\\$1')
}

function replaceAll(str, find, replace) {
  return str.replace(new RegExp(escapeRegExp(find), 'g'), replace)
}

export default {
  name: 'Profile',
  components: { ExampleCard },
  data() {
    return {
      fullscreenLoading: false,
      isDoneSubmit: false,
      resultUrl: '',
    }
  },
  methods: {
    handleSubmit() {
      const { tableData, tableHeader, postForm, actionsList, clientFieldName } = this.$route.params

      const loading = this.$loading({
        lock: true,
        text: 'Loading',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

      // create csv format
      let header = ''
      const rows = {}

      for (let i = 0; i < tableHeader.length; i++) {
        const field = tableHeader[i]
        header += (field + (i == tableHeader.length - 1 ? '\n' : ','))

        for (let j = 0; j < tableData.length; j++) {
          if (rows[j] === undefined) {
            rows[j] = ''
          }
          rows[j] += (tableData[j][field] + (i == tableHeader.length - 1 ? (j == tableData.length - 1 ? '' : '\n') : ','))
        }
      }

      let data = header

      for (const index in rows) {
        // propertyName is what you want
        // you can get the value like this: myObject[propertyName]
        data += rows[index]
      }

      // process split contentTemplate
      //console.log(postForm.content);
      const formatedAndUnFormated = postForm.content.split('#$#@');
      if(formatedAndUnFormated.length == 2) {
        const formated = formatedAndUnFormated[0];
        const unFormated = formatedAndUnFormated[1];

        const regex = /.*Currency Selection.*/g
        const m = regex.exec(formated)
        
        if(m !== null) {
          let newStr = formated
          for (let i = 0; i < m.length; i++) {
            newStr = insert(newStr, m.index, 'SPLITED')
            // + 1 is for new line.
            newStr = insert(newStr, m.index + m[0].length + 'SPLITED'.length + 1, 'SPLITED')
          }

          var splits = newStr.split('SPLITED')
        }

        //console.log(JSON.stringify({unFormated}));
        //console.log(JSON.stringify({unFormated : unFormated.replace('\n\n','\n')}));
        //return;

        axios.post('https://bw5ztr856l.execute-api.ap-southeast-1.amazonaws.com/demo/upload', {
          category: 'DigitalApp',
          name: 'Digital_test_upload_2',
          service: 'MKEDIGITAL',
          serviceKey: 'demomkedigitalservicekey',
          delimiter: ',',
          AccountIdColumnName: clientFieldName,
          data: data,
          contentTemplate: splits,
          contentTemplateStr: unFormated.replace('\n\n','\n'),
        }, { headers: { 'x-api-key': '6EeCO75hTj9LcIKVlJfTJ7qRIyLRgMmiasSCUAKa' }})
          .then((response) => {
            loading.close()
            this.resultUrl = response.data.message;
            this.$notify({
              title: 'Success',
              message: 'Data has been posted',
              type: 'success',
              duration: 4000
            })
            this.isDoneSubmit = true
            console.log(response)
          })
          .catch(function(error) {
            console.log(error)
            loading.close()
          })

      }else {
        alert('some thing went wrong...');
      }
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'avatar',
      'roles'
    ]),
    examples() {
      const { tableData, tableHeader, postForm, actionsList } = this.$route.params
      const returnExamples = []

      const formatedAndUnFormated = postForm.content.split('#$#@');
      if(formatedAndUnFormated.length !== 2) {
        return [];
      }

      const formated = formatedAndUnFormated[0];

      if (formated === '') {
        return []
      }

      const affectFields = {}
      const affectActions = {}

      // find affect field
      for (let i = 0; i < tableHeader.length; i++) {
        if (formated.includes(tableHeader[i])) {
          affectFields[tableHeader[i]] = true
        }
      }

      for (let i = 0; i < actionsList.length; i++) {
        if (formated.includes(actionsList[i])) {
          affectActions[actionsList[i]] = true
        }
      }

      // loop affect fields to apply to text
      // console.log(affectFields);

      for (let i = 0; i < tableData.length; i++) {
        const row = tableData[i]

        let content = formated;

        for (const field in affectFields) {
          // propertyName is what you want
          // you can get the value like this: myObject[propertyName]
          content = replaceAll(content, `{${field}}`, `<strong>${row[field]}</strong>`)
        }

        for (const action in affectActions) {
          // propertyName is what you want
          // you can get the value like this: myObject[propertyName]
          content = replaceAll(content, `[${action}]`,
            `<div class="el-form-item is-required el-form-item--medium">
            <label for="clientFieldName" class="el-form-item__label">
                ${action}
            </label>
            <div class="el-form-item__content">
                <div class="el-select el-select--medium">
                <!---->
                <div class="el-input el-input--medium el-input--suffix">
                <!---->
                <input type="text" readonly="readonly" autocomplete="off" placeholder="Select" class="el-input__inner"><!----><span class="el-input__suffix"><span class="el-input__suffix-inner"><i class="el-select__caret el-input__icon el-icon-arrow-up"></i><!----><!----><!----><!----></span><!----></span><!----></div><div class="el-select-dropdown el-popper" style="display: none; min-width: 193px;"><div class="el-scrollbar" style=""><div class="el-select-dropdown__wrap el-scrollbar__wrap" style="margin-bottom: -15px; margin-right: -15px;"><ul class="el-scrollbar__view el-select-dropdown__list"><!----><li class="el-select-dropdown__item"><span>PE</span></li><li class="el-select-dropdown__item"><span>PE Changed</span></li><li class="el-select-dropdown__item"><span>PE Rank changed</span></li><li class="el-select-dropdown__item"><span>Number of Records</span></li><li class="el-select-dropdown__item"><span>Account</span></li><li class="el-select-dropdown__item"><span>Date</span></li><li class="el-select-dropdown__item"><span>date__stocks_infor</span></li><li class="el-select-dropdown__item"><span>Industry</span></li><li class="el-select-dropdown__item"><span>Nmravgprice</span></li><li class="el-select-dropdown__item"><span>Nmrbuyshare</span></li><li class="el-select-dropdown__item"><span>Nmrvol</span></li><li class="el-select-dropdown__item"><span>pe_compare</span></li><li class="el-select-dropdown__item"><span>pe_compare__stocks</span></li><li class="el-select-dropdown__item"><span>pe_rank</span></li><li class="el-select-dropdown__item"><span>pe_rank__stocks_in</span></li><li class="el-select-dropdown__item"><span>Sector</span></li><li class="el-select-dropdown__item"><span>Security</span></li><li class="el-select-dropdown__item"><span>User Id</span></li></ul></div><div class="el-scrollbar__bar is-horizontal"><div class="el-scrollbar__thumb" style="transform: translateX(0%);"></div></div><div class="el-scrollbar__bar is-vertical"><div class="el-scrollbar__thumb" style="transform: translateY(0%);">
                </div>
            </div>
            </div><!---->
            </div>
            </div><!---->
            </div>
            </div>`)
        }

        returnExamples.push({
          title: postForm.title,
          content: content
        })
      }

      return returnExamples
    }

  },
  created() {
  }
}
</script>

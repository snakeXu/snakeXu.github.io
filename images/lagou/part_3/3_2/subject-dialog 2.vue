/*
 * @Author: zhangmeibiao
 * @Date: 2019-05-16 15:03:53
 * @Last Modified by: zhangmeibiao
 * @Last Modified time: 2020-07-07 15:08:08
 */

<template>
  <div class="subject-dialog">
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      :close-on-press-escape="false"
      :close-on-click-modal="false"
      width="500px"
      :before-close="resetForm">
      <div class="role-add-form">
        <el-form :model="subjectForm"  ref="subject-form" :rules="rules" label-width="100px"  v-if="dialogVisible">
          <el-form-item label="预算行编码" prop="budgetOutsideDetailCode"> 
            <el-input v-model.trim="subjectForm.budgetOutsideDetailCode" :disabled="!!subjectForm.budgetDetailCode" placeholder="请输入" class="form-input" maxlength="64" show-word-limit></el-input>
          </el-form-item>
          <el-form-item label="预算科目" prop="budgetOutsideDetailCode"> 
            <el-input v-model.trim="subjectForm.budgetOutsideDetailCode" :disabled="!!subjectForm.budgetDetailCode" placeholder="请输入" class="form-input" maxlength="64" show-word-limit></el-input>
          </el-form-item>
          <!-- <el-form-item label="科目名称" prop="name">
            <el-input v-model.trim="subjectForm.name" placeholder="请输入" class="form-input" maxlength="50" show-word-limit></el-input>
          </el-form-item>
          <el-form-item label="科目编码" prop="code">
            <el-input v-model.trim="subjectForm.code"  placeholder="请输入" class="form-input" maxlength="30" show-word-limit></el-input>
          </el-form-item> -->
          <el-form-item label="预算金额" prop="budgetAmount" v-if="(tabType === 0) || tabType !== 0">
            <el-input v-model.trim="subjectForm.budgetAmount" placeholder="请输入" class="form-input"></el-input>
          </el-form-item>
          <!-- <el-form-item label="">
            <el-checkbox v-model="materialTypeStatus" @change="changeStatus(1)">绑定物料分类</el-checkbox>
          </el-form-item>
          <el-form-item label="" prop="materialType" >
            <category-cascade-selector
              class="cascade-selector"
              :disabled="!materialTypeStatus"
              @update:selectedIds="categoryIdChange"
              @change="categoryNodeChange"
              :lazy="false"
              :selectedIds="categoryId"
              :multiple="true"
              :width="320"
              :height="40"
              :layerWidth="140"
              :level="2"
              :selectedData="categorySelectedData">
            </category-cascade-selector>
          </el-form-item> -->
          <!-- <el-form-item label="">
            <el-checkbox v-model="mallCategoryStatus" @change="changeStatus(2)">绑定商城类目</el-checkbox>
          </el-form-item>
          <el-form-item label="" prop="mallCategory" >
            <category-cascade-selector
              :disabled="!mallCategoryStatus"
              @update:selectedIds="mallCategoryIdChange"
              @change="mallCategoryNodeChange"
              :lazy="false"
              :selectedIds="mallCategoryId"
              :multiple="true"
              :isMall="true"
              :width="320"
              :height="40"
              :layerWidth="140"
              :level="3"
              :selectedData="mallCategorySelectedData">
            </category-cascade-selector>
          </el-form-item> -->
          <el-form-item label="预算责任人" prop="personCharge" v-if="(tabType === 0) || tabType === 1">
            <el-select
              class="form-input"
              v-model="subjectForm.personCharge"
              multiple
              collapse-tags
              filterable
              remote
              size="small"
              placeholder="支持模糊搜索请输入员工姓名"
              :remote-method="_getPersonList"
              :loading="personLoading">
              <el-option
                v-for="item in personList"
                :key="item.id"
                :label="item.name"
                :value="item.id">
                <span style="float: left">{{ item.name}}</span>
                <span style="float: right; margin-right: 30px;" v-if="item.code">工号：{{ item.code}}</span>
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="备注">
            <el-input
              type="textarea"
              :rows="2"
              placeholder="请输入"
              v-model="subjectForm.remark" maxlength="1000" show-word-limit>
            </el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="default" @click="resetForm">取消</el-button>
        <el-button type="primary" @click="submit">提交</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  import userApi from '@/apis/user' // 0715 删除 预算责任人 删除获取责任人列表
  import response from '@/utils/response' // 0715 删除 预算责任人 删除获取责任人列表
  // import CategoryCascadeSelector from '@/components/category/category-cascade-selector'

  export default {
    name: 'subject-dialog',
    props: {
      // controlType: {
      //   type: Number
      // },
      dialogVisible: {
        type: Boolean,
        default: false
      },
      tabType: {
        type: Number,
        default: 0
      },
      data: {
        type: Object
      }
    },
    data: function () {
      const validatePersonCharge = (rule, value, callback) => {
        if (value.length < 4) {
          callback()
        } else {
          callback(new Error('预算负责人最多只能选择3个'))
        }
      } //0715 删除 预算责任人 相关
      // const validateMaterialType = (rule, value, callback) => {
      //   if (value.length < 11) {
      //     callback()
      //   } else {
      //     callback(new Error('物料分类最多只能选择10个'))
      //   }
      // }
      // const validateMallCategory = (rule, value, callback) => {
      //   if (value.length < 11) {
      //     callback()
      //   } else {
      //     callback(new Error('商城类目最多只能选择10个'))
      //   }
      // }
      return {
        title: '新增条目信息',
        materialTypeStatus: true,
        mallCategoryStatus: true,
        subjectForm: {
          budgetOutsideDetailCode: '', // 预算行编码
          budgetDetailCode: '',
          name: '',
          code: '',
          budgetAmount: '',
          personCharge: [], // 0715 删除 预算责任人相关
          // materialType: [], // 后台类目code
          // mallCategory: [], // 前台类目code
          // materialTypeStatus: 0, // 默认0空数据 1部分数据 2全量数据 
          // mallCategoryStatus: 0, // 默认0空数据 1部分数据 2全量数据
          remark: ''
        },
        isFocus: false,
        // categoryId: [], // 后台类目
        // categorySelectedData: [], // 后台类目选中
        // categorySelectCode: [], // 存放已选中的id，code对应关系
        // mallCategoryId: [], // 前台类目
        // mallCategorySelectedData: [], // 已选中的前台类目
        // mallCategorySelectCode: [], // 已选中的前台类目code
        personLoading: false, // 0715 预算责任人 获取数据是否显示loading
        personList: [], //0715 预算责任人 预算责任人list
        rules: {
          budgetOutsideDetailCode: [
            { required: true, message: '请输入行编码', trigger: ['blur', 'change'] }
          ],
          budgetAmount: [
            { required: true, message: '请输入预算金额', trigger: ['blur', 'change'] },
            { pattern: /^(0|[1-9]\d{0,11})($|\.\d{1,2}$)/, message: '金额格式不正确,整数不能超过12位,小数不能超过2位', trigger: ['blur', 'change'] }
          ],
          name: [
            { required: true, message: '请输入名称', trigger: ['blur', 'change'] },
            { max: 50, message: '最多50个字符', trigger: ['blur', 'change'] }
          ],
          code: [
            { required: true, message: '请输入编码', trigger: ['blur', 'change'] },
            { max: 30, message: '最多30个字符', trigger: ['blur', 'change'] }
          ],
          personCharge: [
            { validator: validatePersonCharge, trigger: ['blur', 'change'] }
          ] //0715 删除 预算责任人 相关
          // materialType: [
          //   { validator: validateMaterialType, trigger: ['blur', 'change'] }
          // ],
          // mallCategory: [
          //   { validator: validateMallCategory, trigger: ['blur', 'change'] }
          // ]
        }
      }
    },
    methods: {
      changeCode () {
        if (this.tabType === 0) {
          this.subjectForm.code = ''
        }
      },
      focusCode () {
        this.isFocus = true
      },
      blurCode () {
        this.isFocus = false
      },
      // keydownCode () {
      //   if (this.tabType === 0) {
      //     console.log(this.$parent.tableData)
      //     let tableData = this.$parent.tableData
      //     let filterArr = tableData.filter((item) => item.code === '*999999')
      //     if (filterArr.length) {
      //       this.$message({
      //         type: 'error',
      //         message: '该科目编码*999999已存在'
      //       })
      //     } else {
      //       this.subjectForm.code = '*999999'
      //       this.subjectForm.name = '不控制金额科目'
      //       this.subjectForm.budgetAmount = ''
      //     }
      //   }
      // },
      resetForm () {
        this.subjectForm = Object.assign({
          // subjectForm: '',
          budgetOutsideDetailCode: '',
          name: '',
          code: '',
          budgetAmount: '',
          personCharge: [], //预算责任人相关
          // materialType: [],
          // mallCategory: [],
          remark: ''
        })
        // this.categoryId = []
        // this.categorySelectedData = []
        // this.mallCategoryId = []
        // this.mallCategorySelectedData = []
        // this.materialTypeStatus = true
        // this.mallCategoryStatus = true
        if (this.$refs['subject-form']) {
          this.$refs['subject-form'].resetFields()
        }
        this.$emit('close')
      },
      // changeStatus (type) {
      //   // 后台类目
      //   if (type === 1 && !this.materialTypeStatus) {
      //     this.categoryId = []
      //     this.categorySelectedData = []
      //     this.categorySelectCode = []
      //     this.subjectForm.materialType = []
      //     this.subjectForm.materialTypeStatus = 0
      //   }
      //   // 前台类目
      //   if (type === 2 && !this.mallCategoryStatus) {
      //     this.mallCategoryId = []
      //     this.mallCategorySelectedData = []
      //     this.mallCategorySelectCode = []
      //     this.subjectForm.mallCategory = []
      //     this.subjectForm.mallCategoryStatus = 0
      //   }
      // },
      submit () {
        let self = this
        this.$refs['subject-form'].validate((valid) => {
          if (valid) {
            let subjectForm = Object.assign({}, self.subjectForm)
            // 0715需求 删除预算责任人相关
            // 个人预算没有责任人
            if (self.tabType === 2) {
              delete subjectForm.personCharge
            } else {
              subjectForm.personCharge = JSON.stringify(self.subjectForm.personCharge)
            }
            // 后台类目状态类型
            // if (!self.materialTypeStatus) {
            //   subjectForm.materialTypeStatus = 0
            // } else {
            //   if (subjectForm.materialType.length && self.materialTypeStatus) {
            //     subjectForm.materialTypeStatus = 1
            //   } else {
            //     subjectForm.materialTypeStatus = 2
            //   }
            // }
            // 前台类目类型
            // if (!self.mallCategoryStatus) {
            //   subjectForm.mallCategoryStatus = 0
            // } else {
            //   if (subjectForm.mallCategory.length && self.mallCategoryStatus) {
            //     subjectForm.mallCategoryStatus = 1
            //   } else {
            //     subjectForm.mallCategoryStatus = 2
            //   }
            // }
            // subjectForm.materialType = JSON.stringify(self.subjectForm.materialType)
            // subjectForm.mallCategory = JSON.stringify(self.subjectForm.mallCategory)
            console.log('submit')
            console.log(subjectForm)
            self.$emit('submitSubject', subjectForm)
          } 
        })
      },
      // // 后台分类改变
      // categoryIdChange (selectIds) {
      //   this.categoryId = selectIds
      //   for (let i = this.categorySelectCode.length - 1; i >= 0; i--) {
      //     if (this.categoryId.indexOf(this.categorySelectCode[i].id) === -1) {
      //       let deleteCode = this.categorySelectCode[i].code
      //       this.categorySelectCode.splice(i, 1)
      //       for (let i = this.subjectForm.materialType.length - 1; i >= 0; i--) {
      //         if (this.subjectForm.materialType[i] === deleteCode) {
      //           this.subjectForm.materialType.splice(i, 1)
      //         }
      //       }
      //     }
      //   }
      // },
      // // 前台分类改变
      // mallCategoryIdChange (selectIds) {
      //   console.log('111', selectIds)
      //   console.log(this.mallCategorySelectCode)
      //   console.log(this.mallCategoryId)
      //   console.log(this.subjectForm.mallCategory)
      //   this.mallCategoryId = selectIds
      //   for (let i = this.mallCategorySelectCode.length - 1; i >= 0; i--) {
      //     if (this.mallCategoryId.indexOf(this.mallCategorySelectCode[i].id) === -1) {
      //       let deleteCode = this.mallCategorySelectCode[i].code
      //       this.mallCategorySelectCode.splice(i, 1)
      //       for (let i = this.subjectForm.mallCategory.length - 1; i >= 0; i--) {
      //         if (this.subjectForm.mallCategory[i] === deleteCode) {
      //           this.subjectForm.mallCategory.splice(i, 1)
      //         }
      //       }
      //     }
      //   }
      // },
      // // 后台分类节点改变
      // categoryNodeChange (node) {
      //   if (node.simple && Array.isArray(node.simple) && node.simple.length) {
      //     node.simple.map((childNode) => {
      //       if (this.subjectForm.materialType.indexOf(childNode.code) < 0) {
      //         this.subjectForm.materialType.push(childNode.code)
      //         this.categorySelectCode.push({
      //           id: childNode.id,
      //           code: childNode.code
      //         })
      //       }
      //     })
      //   }
      //   console.log(this.subjectForm.materialType)
      // },
      // // 前台分类改变
      // mallCategoryNodeChange (node) {
      //   console.log('222', node)
      //   if (node.simple && Array.isArray(node.simple) && node.simple.length) {
      //     node.simple.map((childNode) => {
      //       if (this.subjectForm.mallCategory.indexOf(childNode.systemCode) < 0) {
      //         this.subjectForm.mallCategory.push(childNode.systemCode)
      //         this.mallCategorySelectCode.push({
      //           id: childNode.id,
      //           code: childNode.systemCode
      //         })
      //       }
      //     })
      //   }
      //   console.log(this.subjectForm.mallCategory)
      // },
      // deleteCategory (node) {
      //   console.log(node)
      //   let code = ''
      //   this.categorySelectCode.map(item => {
      //     if (node.data) {
      //       if (item.id === node.data.id) {
      //         code = item.code
      //       }
      //     } else {
      //       if (item.id === node.id) {
      //         code = item.code
      //       }
      //     }
      //   })
      //   if (code) {
      //     for (let i = this.subjectForm.materialType.length - 1; i >= 0; i--) {
      //       if (this.subjectForm.materialType[i] === code) {
      //         this.subjectForm.materialType.splice(i, 1)
      //       }
      //     }
      //   }
      //   console.log(this.subjectForm.materialType)
      // },
      // 获取用户列表 //0715 删除预算责任人 删除获取责任人列表
      async _getPersonList (query) {
        const api = userApi.searchName
        let params = Object.assign({name: query})
        this.personLoading = true
        const res = await response(this.$http(api.type, api.url, params), {loadingOptions: false})
        if (res.isOk) {
          this.personList = res.content || []
        }
        this.personLoading = false
      }
    },
    components: {
      // CategoryCascadeSelector
    },
    created () {
      this._getPersonList() //0715 删除预算责任人 删除获取责任人列表
    },
    watch: {
      data: {
        handler (val, oldVal) {
          if (val) {
            this.subjectForm.budgetDetailCode = val.budgetDetailCode
            if (this.subjectForm.budgetDetailCode) {
              this.title = '编辑条目信息'
            }
            this.subjectForm.budgetOutsideDetailCode = val.budgetOutsideDetailCode
            this.subjectForm.name = val.name
            this.subjectForm.code = val.code
            this.subjectForm.budgetAmount = val.budgetAmount
            this.subjectForm.materialType =  JSON.parse(val.materialType)
            this.subjectForm.mallCategory = JSON.parse(val.mallCategory)
            // this.materialTypeStatus = Boolean(val.materialTypeStatus)
            // this.mallCategoryStatus = Boolean(val.mallCategoryStatus)
            // this.subjectForm.materialTypeStatus = val.materialTypeStatus
            // this.subjectForm.mallCategoryStatus = val.mallCategoryStatus
            this.subjectForm.remark = val.remark
            // this.categoryId = []
            // this.categorySelectedData = []
            // this.categorySelectCode = []
            // this.mallCategoryId = []
            // this.mallCategorySelectedData = []
            // this.mallCategorySelectCode = []
            // // 后台类目
            // if (Array.isArray(val.materialTypeList)) {
            //   val.materialTypeList.map((item) => {
            //     this.categoryId.push(item.id)
            //     this.categorySelectedData.push({id: item.id, name: item.name})
            //     this.categorySelectCode.push({id: item.id, code: item.code})
            //   })
            // }
            // // 商城类目
            // if (Array.isArray(val.mallCategoryList)) {
            //   val.mallCategoryList.map((item) => {
            //     console.log('ppppppppp', item)
            //     this.mallCategoryId.push(item.id)
            //     this.mallCategorySelectedData.push({id: item.id, name: item.name})
            //     this.mallCategorySelectCode.push({id: item.id, code: item.code})
            //   })
            // }
            // 0715 预算责任人 相关
            if (this.tabType !== 2) {
              this.subjectForm.personCharge = JSON.parse(val.personCharge)  
              for (let i = 0; i < val.personChargeList.length; i++) {
                let item = val.personChargeList[i]
                let isHas = false
                for (let j = 0; j < this.personList.length; j++) {
                  let person = this.personList[j]
                  if (item.id === person.id) {
                    isHas = true
                  }
                }
                if (!isHas) {
                  this.personList.push({
                    id: item.id,
                    code: item.code,
                    name: item.name
                  })
                }
              }
            }
          } else {
            this.title = '新增条目信息'
          }
        },
        immediate: true
      }
      // controlType (val) {
      //   if (val === 10 || val === 11 || val === 20 || val === 30) {
      //     this.$set(this.rules, 'budgetAmount', [
      //       { required: true, message: '请输入预算金额', trigger: ['blur', 'change'] },
      //       { pattern: /^(0|[1-9]\d{0,11})($|\.\d{1,2}$)/, message: '金额格式不正确,整数不能超过12位,小数不能超过2位', trigger: ['blur', 'change'] }
      //     ])
      //   } else {
      //     this.$set(this.rules, 'budgetAmount', [
      //       { pattern: /^(0|[1-9]\d{0,11})($|\.\d{1,2}$)/, message: '金额格式不正确,整数不能超过12位,小数不能超过2位', trigger: ['blur', 'change'] }
      //     ])
      //   }
      // }
    }
  }
</script>

<style lang="scss" scoped>
  .subject-dialog {
    .form-input{
      width: 320px;
    }
  }
</style>
<style lang="scss">
.subject-dialog {
   .cascade-selector {
    .sy-tc-selector{
      z-index: 101;
    }
  }
}
</style>

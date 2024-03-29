<template>
  <div class="user-manage">
    <div class="query-form">
      <el-form inline :model="user" ref="ruleFormRef">
        <el-form-item label="用户ID" prop="userId">
          <el-input v-model="user.userId" placeholder="请输入用户ID"></el-input>
        </el-form-item>
        <el-form-item label="用户名称" prop="userName">
          <el-input v-model="user.userName" placeholder="请输入用户名称"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="state">
          <el-select v-model="user.state">
            <el-option :value="0" label="全部"></el-option>
            <el-option :value="1" label="在职"></el-option>
            <el-option :value="2" label="离职"></el-option>
            <el-option :value="3" label="试用期"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleQuery">查询</el-button>
          <el-button @click="handleReset">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="base-table">
      <div class="action">
        <el-button type="primary" @click="handleAddForm">新增</el-button>
        <el-button type="danger" @click="handleBatchDelete">批量删除</el-button>
        <el-button style="margin-left: auto" type="primary" @click="handleExportExcel">导出表格</el-button>
      </div>
      <el-table
          :data="userList"
          border
          stripe
          :height="tableHeight"
          tooltip-effect="dark"
          style="width: 100%;"
          class="el-table-box"
          @selection-change="handleSelectionChange"
      >
        <el-table-column
            type="selection"
            align="center"
            width="55"/>
        <el-table-column
            :prop="item.prop"
            :label="item.label"
            :width="item.width ? item.width : 'auto'"
            v-for="(item,index) in columns"
            :key="index"
            :align="item.align"
            :formatter="item.formatter"
        />
        <el-table-column
            fixed="right"
            align="center"
            label="操作"
            width="150">
          <template #default="scope">
            <el-button type="primary" size="small" @click="handleEdit(scope.row)">编辑</el-button>
            <el-button type="danger" size="small" @click="handleDelete(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
            background
            :page-size="pager.pageSize"
            layout="total, prev, pager, next"
            :total="pager.total"
            @current-change="handleCurrentPageChange"
        />
      </div>
    </div>
    <!-- 弹窗 -->
    <el-dialog v-model="userFromModalShow" title="新增用户数据">
      <el-form
          :model="userForm"
          label-width="100px"
          label-position="left"
          ref="userFormRef"
          :rules="rules"
      >
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="userForm.userName" placeholder="请输入用户名称"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="userEmail">
          <el-input v-model="userForm.userEmail" placeholder="请输入邮箱">
            <template #append>@yam.com</template>
          </el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="userForm.mobile" placeholder="请输入手机号"></el-input>
        </el-form-item>
        <el-form-item label="岗位" prop="job">
          <el-input v-model="userForm.job" placeholder="请输入岗位"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="state">
          <el-radio-group v-model="userForm.state" style="width: 100%">
            <el-radio border :label="1">在职</el-radio>
            <el-radio border :label="2">离职</el-radio>
            <el-radio border :label="3">试用期</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="系统角色" prop="roleList">
          <el-select
              v-model="userForm.roleList"
              style="width: 100%"
              placeholder="请选择对应角色"
              multiple
              clearable
              filterable
          >
            <el-option
                v-for="role in roleList"
                :key="role._id"
                :label="role.roleName"
                :value="role._id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="所属部门" prop="deptId">
          <el-cascader
              v-model="userForm.deptId"
              style="width: 100%"
              :options="deptList"
              placeholder="请选择所属部门"
              :props="{checkStrictly : true,value:'_id',label:'deptName'}"
              clearable
              filterable
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button type="primary" @click="handleSubmitForm()">确认</el-button>
        <el-button @click="handleResetForm()">重置</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {
  ref,
  reactive,
  onMounted,
  getCurrentInstance,
  toRaw,
  nextTick,
  computed
} from 'vue'
import dayjs from "dayjs";
import {ElMessage} from "element-plus";
import exportExcel from "../common/exportExcel"
import _ from 'lodash'

// 使用 $ref 可以直接用定义, 而不需要用 value 来改变
// let count = $ref(0)
// console.log(count);  // 无value 包裹
// console.log($$(count)); // 有 value 包裹
// let reactive1 = $ref({   // 可以直接写数组
//   a: 1, b: 2, c: 3
// })
// console.log(reactive1);  // 原始
// console.log(reactive1.a); // 获取
// reactive1.b = 100        // 更改
// console.log(reactive1.b);  // 获取
// console.log($$(reactive1)); // 封装 proxy

onMounted(async () => {
  await getUserList() // 获取用户列表
  await getRoleList() // 获取角色列表
  await getDeptList() // 获取部门列表
  await getAutoHeight() // 动态计算 table 高度
})

// 用户表单
let user = $ref({
  userId: '',
  userName: '',
  state: 0
})

// table 高度
let tableHeight = $ref(100)
// 动态计算高度
const getAutoHeight = () => {
  let el = document.querySelector(".user-manage")
  let form = document.querySelector(".query-form")
  let action = document.querySelector(".action")
  nextTick(() => {
    // 整体高度 - action 和 form 的高度, 并且减去padding  120 px
    tableHeight = el.parentNode.clientHeight - action.clientHeight - form.clientHeight - 120
  });
}

let flag = $ref(true)
// 窗口发生改变, 重新计算 table 高度
window.onresize = () => {
  if (!flag) return
  getAutoHeight()
  flag = false;
  setTimeout(() => {
    flag = true;
  }, 100)
}


// 用户 table 表头
let columns = $ref([
  {
    prop: 'userId',
    label: '用户ID',
    align: 'center'
  },
  {
    prop: 'userName',
    label: '用户名称',
    align: 'center'
  },
  {
    prop: 'userEmail',
    label: '用户邮箱',
    width: '200'
  },
  {
    prop: 'role',
    label: '用户角色',
    align: 'left',
    formatter: (row, column, value) => {
      // console.log(row);
      // console.log(column);
      // console.log(value);
      // return value === 0 ? '管理员' : '普通用户'
      return {
        0: '管理员',
        1: '普通用户'
      }[value]
    }
  },
  {
    prop: 'state',
    label: '用户状态',
    align: 'center',
    formatter: (row, column, value) => {
      return {
        1: '在职',
        2: '离职',
        3: '试用期'
      }[value]
    }
  },
  {
    prop: 'createTime',
    label: '注册时间',
    width: '170',
  },
  {
    prop: 'lastLoginTime',
    label: '最后登录时间',
    width: '170',
  },
])
// 用户 table 内容
let userList = $ref([])
// 分页器
let pager = $ref({
  pageNum: 1, // 当前页码
  pageSize: 10, // 每页条数
  total: 20   // 总条数 (动态传入)
})
// 定义用户操作行为 (新增 / 编辑) 数据
let action = $ref('add')
// 批量删除勾选的列表
let batchDeleteList = $ref([]) // 批量删除 id

// 其中包含了 main.js 里面的 config.globalProperties
// const {proxy} = getCurrentInstance()
// prod(生产环境下) 模式下, 不会有 config.globalProperties
// const {ctx} = getCurrentInstance()
// console.log(getCurrentInstance());


// 删除单条或者多条table数据
const handleDelete = async (row) => {
  try {
    let res = await $api.postUserDelete({
      userIds: [row.userId]
    })
    if (res.nModified > 0) {
      ElMessage.success('删除成功')
      batchDeleteList = []
      await getUserList()
    } else {
      ElMessage.error('删除失败')
    }
  } catch (e) {
    console.log(e);
  }
}

// 处理批量删除
const handleBatchDelete = async () => {
  if (batchDeleteList.length === 0) {
    ElMessage.warning('请选择要删除的数据')
    return
  }
  try {
    let res = await $api.postUserDelete({
      userIds: batchDeleteList
    })
    if (res.nModified > 0) {
      ElMessage.success('删除成功')
      batchDeleteList = []
      await getUserList()
    } else {
      ElMessage.error('删除失败')
    }
  } catch (e) {
    console.log(e);
  }
}

// 处理更改当前页码
const handleCurrentPageChange = (val) => {
  // console.log(val);
  pager.pageNum = val
  getUserList()
}

// const handlePageSizeChange = () => {
//   console.log(456); // 更改所需的页码
// }

// 获取用户列表数据
const getUserList = async () => {
  // 传参
  let params = {...user, ...pager}
  // console.log(params);
  try {
    // let {list, page} = await proxy.$api.getUserList()
    let {list, page} = await $api.getUserList(params) // 对应的 $api 会自动找 window 对象
    list = list.map(item => {
      return {
        ...item,
        createTime: dayjs(item.createTime).format('YYYY-MM-DD HH:mm'),
        lastLoginTime: dayjs(item.lastLoginTime).format('YYYY-MM-DD HH:mm')
      }
    })
    // console.log(list);
    // console.log(page);
    userList = list
    pager.total = page.total
    // pager.pageNum = page.pageNum
  } catch (e) {
    console.log(e);
  }
}

// 处理 select 选中
const handleSelectionChange = (row) => {
  // console.log(row);
  batchDeleteList = []
  row.forEach(item => {
    batchDeleteList.push(item.userId)
  })
  // console.log(batchDeleteList);
}

// 查询 , 获取用户列表
const handleQuery = () => {
  // console.log('查询');
  getUserList()
}

// ref 可以获取标签中的 ref , 但是必须要在外层定义, 而不能在方法中定义
const ruleFormRef = $ref(null)
// 重置 , 重置查询条件
const handleReset = () => {
  ruleFormRef && ruleFormRef.resetFields()
}

// 新增用户表单 Form dialog
const userForm = $ref({
  userName: '',
  userEmail: '',
  mobile: '',
  job: '',
  state: 3,
  roleList: '',
  deptId: ''
})

const rules = $ref({
  userName: [
    {required: true, message: '请输入名称', trigger: 'blur'},
    {min: 2, max: 5, message: '请输入3-5位数的名称', trigger: 'blur'},
  ],
  userEmail: [
    {required: true, message: '请输入邮箱', trigger: 'blur'},
  ],
  mobile: [
    {pattern: /1\d{10}/, message: '请输入正确的手机号 ', trigger: 'blur'},
  ]
})


// Form dialog 显示和隐藏
let userFromModalShow = $ref(false)


// 新增 Form 用户表单
const handleAddForm = () => {
  handleResetForm() // 重置表单
  userFromModalShow = true
  action = 'add'
}

// 获取 userForm ref
const userFormRef = $ref(null)

// 提交 Form 用户表单
const handleSubmitForm = async () => {
  // 1. 校验表单
  if (!userFormRef) return
  await userFormRef.validate(async (valid, fields) => {
    if (valid) {
      // userForm.userEmail += "@yam.com" // 会篡改元数据
      let userEmail = toRaw(userForm.userEmail) // 防止篡改直接数据
      userEmail += "@yam.com"
      let params = {
        ...toRaw(userForm),
        userEmail,
        action
      }
      // 2. 提交成功操作
      let res = await $api.postUserSubmit(params)
      if (res) {
        let msg = action === "add" ? '新增' : '编辑'
        ElMessage.success(msg + '成功')
        handleResetForm() // 重置表单
        userFromModalShow = false // 关闭弹框
        await getUserList() // 刷新列表
      }
    } else {
      console.log('error submit!', fields)
      ElMessage.error('请检查输入内容是否正确')
    }
  })

}

// 重置 Form 用户表单
const handleResetForm = () => {
  userFormRef && userFormRef.resetFields()
}

// 编辑表格某一条数据
const handleEdit = (row) => {
  action = "edit"
  // console.log(row);
  userFromModalShow = true // 打开form弹框
  nextTick(() => {
    Object.assign(userForm, row)
  })
}

// 导出表格
const handleExportExcel = () => {
  let fields = {}
  columns.forEach(item => {
    fields[item.prop] = item.label
  })
  // 深拷贝数据 1
  // let tableData = JSON.parse(JSON.stringify(userList));
  // 深拷贝数据 2
  let tableData = _.cloneDeep(userList)
  exportExcel(tableData, fields, "用户人员名单");
}

let deptList = $ref([])
// 获取部门列表
const getDeptList = async () => {
  deptList = await $api.getDeptList()
}


let roleList = $ref([])
// 获取角色列表
const getRoleList = async () => {
  roleList = await $api.getRoleList()
}

</script>

<style scoped lang="scss">
</style>

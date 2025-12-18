<template>
  <el-card class="card">
    <h1>我提交的bug列表</h1>
    <el-input
      v-model="data.tableData.title"
      class="input1"
      placeholder="请输入BUG名称查询"
      prefix-icon="Search"
    />
    <el-button type="primary" @click="load">搜索</el-button>
    <div class="action-group">
<!--      <el-button type="success" @click="">新增</el-button>-->
      <el-button type="danger" @click="delBatch"> 批量删除 </el-button>
<!--      <el-button type="info">导入</el-button>-->
      <el-button type="primary">导出</el-button>
    </div>


    <!-- 表格 -->
    <el-table
        :data="data.tableData"
        stripe
        :default-sort="{ prop: 'create_time', order: 'descending' }"
        style="width: 100%; margin-top: 16px"
        @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" />

      <el-table-column prop="title" label="BUG名称" show-overflow-tooltip width="120" />
      <el-table-column prop="severity" label="严重度" width="180" >
        <template #default="scope">
          <span  v-if="scope.row.severity === 1">一般</span>
          <span v-else>
      {{scope.row.severity === 2 ? '严重' :
              scope.row.severity === 3 ? '致命':'无'
            }}
    </span>
        </template>
      </el-table-column>
      <el-table-column prop="priority" label="BUG优先级" width="180" >
        <template #default="scope">
          <span  v-if="scope.row.severity === 1">低</span>
          <span v-else>
      {{scope.row.severity === 2 ? '中' :
              scope.row.severity === 3 ? '高':'无'
            }}
    </span>
        </template>
      </el-table-column>
      <el-table-column prop="module" label="所属模块" width="180" >
      <template #default="scope">
        <span v-if="scope.row.module === 1">前端界面</span>
        <span v-else>
      {{scope.row.module === 2 ? '后端逻辑' :
            scope.row.module === 3 ? '数据库':
                scope.row.module === 4 ? '其他':'无'
          }}
    </span>
      </template>
      </el-table-column>
      <el-table-column label="创建时间" prop="create_time" width="200">
        <template #default="scope">
          {{ formatTime(scope.row.create_time) }}
        </template>
      </el-table-column>
      <el-table-column prop="remark" show-overflow-tooltip label="复现步骤" />
      <el-table-column prop="document" show-overflow-tooltip label="附件" />
      <el-table-column label="操作" width="300px">
        <template #default="scope">
          <el-button
              size="small"
              type="primary"
              :icon="Postcard"
              @click="goToBugDetail(scope.row.id)"
          >
            详细
          </el-button>
          <el-button
              size="small"
              type="primary"
              :icon="Edit"
              @click="openEdit(scope.row)"
          >
            编辑
          </el-button>
          <el-button
              size="small"
              type="danger"
              :icon="DeleteFilled"
              @click="deleteBug(scope.row.id)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

<!--     分页-->
    <div style="margin-top: 10px">
      <el-pagination
          @current-change="load"
          @size-change="load"
          v-model:current-page="data.pageNum"
          v-model:page-size="data.pageSize"
          :page-sizes="[2, 10, 15, 20]"
          background
          layout="total, sizes, prev, pager, next, jumper"
          :total="data.total"
      />
    </div>
<!--     编辑弹窗-->
<!--    :title="data.dialogType === 'add' ? '新增' : '编辑'"-->
    <el-dialog
        title="编辑BUG信息"
        v-model="data.dialogVisible"
        width="500px"
        :close-on-click-modal="false"
    >
      <el-form
          ref="dataFormRef"
          :model="data.bug_list"
      :rules="data.formRules"
      label-width="100px"
      status-icon
      >
      <!-- 标题：对应rules中的title规则 -->
      <el-form-item label="标题" prop="title">  <!-- prop必须与rules中的key一致 -->
        <el-input v-model="data.bug_list.title" placeholder="请输入标题" />
      </el-form-item>
      <!-- 严重度：对应rules中的severity规则 -->
        <el-form-item label="严重度" prop="severity">
          <el-select v-model="data.bug_list.severity" placeholder="请选择严重度"
          >
          <el-option label="一般" :value=1 />
          <el-option label="严重" :value=2 />
          <el-option label="致命" :value=3 />
          </el-select>
        </el-form-item>

      <!-- 优先级：对应rules中的priority规则 -->
      <el-form-item label="优先级" prop="priority">
        <el-select v-model="data.bug_list.priority" placeholder="请选择优先级">
          <el-option label="低" :value=1 />
          <el-option label="中" :value=2 />
          <el-option label="高" :value=3 />
        </el-select>
      </el-form-item>

      <!-- 所属模块：对应rules中的module规则 -->
        <el-form-item label="所属模块" prop="module">
          <el-select v-model="data.bug_list.module" placeholder="请选择模块"
          >
            <el-option label="前端界面 " :value=1 />
            <el-option label="后端逻辑" :value=2 />
            <el-option label="数据库" :value=3 />
            <el-option label="其他" :value=3 />
          </el-select>
        </el-form-item>

      <!-- 备注：非必填，可不加校验 -->
      <el-form-item label="备注" prop="remark">
        <el-input v-model="data.bug_list.remark" placeholder="请输入备注" type="textarea" />
      </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="data.dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="save">确认</el-button>
      </template>
    </el-dialog>

<!--    文件上传-->
<!--    <el-upload-->
<!--        v-model:file-list="fileList"-->
<!--        class="upload-demo"-->
<!--        action="/api/upload" &lt;!&ndash; 替换为你的后端上传接口 &ndash;&gt;-->
<!--    :on-preview="handlePreview"-->
<!--    :on-remove="handleRemove"-->
<!--    :on-success="handleSuccess" &lt;!&ndash; 新增：上传成功回调 &ndash;&gt;-->
<!--    :on-error="handleError" &lt;!&ndash; 新增：上传失败回调 &ndash;&gt;-->
<!--    :before-upload="beforeUpload" &lt;!&ndash; 新增：上传前校验 &ndash;&gt;-->
<!--    list-type="picture"-->
<!--    accept="image/jpeg,image/png" &lt;!&ndash; 限制仅上传jpg/png &ndash;&gt;-->
<!--    :limit="3" &lt;!&ndash; 新增：最多上传3个文件 &ndash;&gt;-->
<!--    :on-exceed="handleExceed" &lt;!&ndash; 新增：超过数量限制时的回调 &ndash;&gt;-->
<!--    >-->
<!--    <el-button type="primary">点击上传图片</el-button>-->
<!--    <template #tip>-->
<!--      <div class="el-upload__tip">-->
<!--        仅支持 jpg/png 格式，文件大小不超过 500kb，最多上传 3 个文件-->
<!--      </div>-->
<!--    </template>-->
<!--    </el-upload>-->


  </el-card>

</template>

<script setup>
import {ref, reactive, onMounted, nextTick, watchEffect} from "vue";
import axios from "@/utils/axios.js";
import {ElMessage, ElMessageBox} from "element-plus";
import dayjs from 'dayjs';
import {DeleteFilled, Edit, Postcard} from "@element-plus/icons-vue";
import router from "@/router/index.js"; // 引入 dayjs
import { jwtDecode } from 'jwt-decode';
const data = reactive({
  id:0,
  dialogType:'',
  deleteDialogVisible:false,
  dialogVisible:false,
  tableData:[],
  bug_list: {
    title: '',
    severity: null ,
    priority: null ,
    module: null ,
    remark: '',
    document: '' ,// 附件路径，提交时填充
    user_id:0
  },
  ids:[],//多选
  pageNum: 1,
  pageSize: 10,
  total: 0,
  formRules:{
    title: [
      { required: true, message: '请输入Bug标题', trigger: 'blur' }
    ],
    severity: [
      { required: true, message: '请选择严重度', trigger: 'change' }
    ],
    priority: [
      { required: true, message: '请选择优先级', trigger: 'change' }
    ],
    module: [
      { required: true, message: '请选择所属模块', trigger: 'change' }
    ]
  }
});

const formatTime = (isoTime) => {
  if (!isoTime) return '-'; // 空值显示为短横线
  // 转换为 "YYYY-MM-DD HH:mm:ss" 格式
  return dayjs(isoTime).format('YYYY-MM-DD HH:mm:ss');
};

onMounted(() => {
  const token = localStorage.getItem('xm-pro-user'); // 假设从本地存储获取
  const user = jwtDecode(token);
  data.id =user.claims.id
  load();
});
// watchEffect(() => {
//   const token = localStorage.getItem('xm-pro-user'); // 假设从本地存储获取
//   try {
//     const user = jwtDecode(token);
//     data.bug_list.user_id = user.claims.id || 0; // 按实际路径修改
//   } catch (err) {
//     console.error('token更新后解析失败:', err);
//   }
// })
// 查询
const load = () => {

  // const user= JSON.parse(localStorage.getItem("xm-pro-user") || "null");
  axios.post("/bugs/query", {
    pageNum:data.pageNum,
    pageSize:data.pageSize,
    user_id: data.id || null,
    title: data.tableData.title || null
  })
      .then(res => {
        if (res.code === 0) {

          data.tableData = res.data.list;
          data.total = res.data.total;
        } else {
          ElMessage.error("查询失败：" + res.message);
        }
      })
      .catch(err => {
        console.error("查询接口异常：", err);
        ElMessage.error("网络错误，请稍后重试");
      })
};

const dataFormRef = ref(null); // 表单引用，用于触发校验
// 保存编辑
const save = () => {
  dataFormRef.value.validate((valid) => {
    if (valid) {
      data.bug_list.user_id = data.id;
      axios.put("/bugs/edit", data.bug_list)
          .then(res => {
            if (res.code === 200) {
              load();
              ElMessage.success("编辑成功");
              data.dialogVisible = false; // 关闭弹窗
            } else {
              ElMessage.error(`编辑失败：${res.message || '未知错误'}`);
            }
          })
          .catch(err => {
            console.error("编辑接口异常：", err);
            ElMessage.error("网络错误，请稍后重试");
          });
    } else {
      ElMessage.error("表单填写有误，请检查后重试");
    }
  });
};
// 打开编辑
const openEdit = (row) => {
  data.bug_list = JSON.parse(JSON.stringify(row));
  data.dialogVisible = true;
}
// 打开单个删除
const deleteBug = (id) => {
  ElMessageBox.confirm('你确定要删除该数据吗?', '确定删除', {type: 'warning'})
      .then(()=>{
    axios.delete('/bugs/deleteId/'+id)
        .then(res=>{
          if (res.code === 200 ) {
            load();
            ElMessage.success("删除成功");
          }else {
            ElMessage.error(`删除成功：${res.message || '未知错误'}`);
          }
        })
  })
      .catch(() => {
        ElMessage({
          type: 'info',
          message: '已取消操作',
        })
      })
}
// 获取多个id用删除
const handleSelectionChange = (rows) => {
  data.ids = rows.map(row=> row.id)
  console.log(data.ids)
}
// 打开多个删除
const delBatch = () => {
  if (data.ids.length ===0){
    ElMessage.warning("请选择数据")
    return
  }
  ElMessageBox.confirm(
      '你确定要删除多个数据吗?', '确定删除', {type: 'warning'}).then(()=>{
    axios.delete('/bugs/deleteIds',{data:data.ids})
        .then(res=>{
          if (res.code === 200 ) {
            load();
            ElMessage.success("删除成功");

          }else {
            ElMessage.error(`删除成功：${res.message || '未知错误'}`);
          }
        })
  })
      .catch(() => {
        ElMessage({
          type: 'info',
          message: '已取消操作',
        })
      })
}

const goToBugDetail = (bugId) => {
  router.push(`/bugmana/bug_detail/${bugId}`);
};
</script>

<style>
.card {
  max-width: 100%;
  margin-bottom: 3px;
}
.input1 {
  width: 240px;
  margin-right: 20px;
}
.data-table-container {
  padding: 20px;
  background-color: #fff;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.08);
}

.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 16px;
}

.search-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.search-input {
  width: 300px;
}

.action-group {
  margin-top: 20px;
  display: flex;
  gap: 10px;
}

.pagination-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 16px;
  padding: 10px 0;
}

.total-count {
  color: #606266;
  font-size: 14px;
}

.page-size-select {
  margin: 0 10px;
}
</style>

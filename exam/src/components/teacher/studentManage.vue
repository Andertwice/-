// 学生管理页面
<template>
  <div class="all">
    <div class="searchBox">
      <input type="text" placeholder="学生姓名" class="search" v-model="key" />
      <el-button type="primary" @click="search()">搜索试卷</el-button>
      <el-button type="primary" @click="downloadFile()">导出学生数据</el-button>
    </div>
    <el-table :data="pagination.records" border>
      <el-table-column
        fixed="left"
        prop="studentName"
        label="姓名"
        width="180"
      ></el-table-column>
      <el-table-column
        prop="institute"
        label="学院"
        width="200"
      ></el-table-column>
      <el-table-column prop="major" label="专业" width="200"></el-table-column>
      <el-table-column prop="grade" label="年级" width="200"></el-table-column>
      <el-table-column prop="clazz" label="班级" width="100"></el-table-column>
      <el-table-column prop="sex" label="性别" width="120"></el-table-column>
      <el-table-column
        prop="tel"
        label="联系方式"
        width="120"
      ></el-table-column>
      <el-table-column fixed="right" label="操作" width="150">
        <template slot-scope="scope">
          <el-button
            @click="checkGrade(scope.row.studentId)"
            type="primary"
            size="small"
            >编辑</el-button
          >
          <el-button
            @click="deleteById(scope.row.studentId)"
            type="danger"
            size="small"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagination.current"
      :page-sizes="[6, 10]"
      :page-size="pagination.size"
      layout="total, sizes, prev, pager, next, jumper"
      :total="pagination.total"
      class="page"
    >
    </el-pagination>
    <!-- 编辑对话框-->
    <el-dialog
      title="编辑学生信息"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <section class="update">
        <el-form ref="form" :model="form" label-width="80px">
          <el-form-item label="姓名">
            <el-input v-model="form.studentName"></el-input>
          </el-form-item>
          <el-form-item label="学院">
            <el-input v-model="form.institute"></el-input>
          </el-form-item>
          <el-form-item label="专业">
            <el-input v-model="form.major"></el-input>
          </el-form-item>
          <el-form-item label="年级">
            <el-input v-model="form.grade"></el-input>
          </el-form-item>
          <el-form-item label="班级">
            <el-input v-model="form.clazz"></el-input>
          </el-form-item>
          <el-form-item label="性别">
            <el-input v-model="form.sex"></el-input>
          </el-form-item>
          <el-form-item label="电话号码">
            <el-input v-model="form.tel"></el-input>
          </el-form-item>
        </el-form>
      </section>
      <span slot="footer" class="dialog-footer">
        <el-button @click="resetPwd()">重置该学生密码</el-button>
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submit()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
var XLSX = require("xlsx");
export default {
  data() {
    return {
      pagination: {
        //分页后的考试信息
        current: 1, //当前页
        total: null, //记录条数
        size: 6, //每页条数
      },
      dialogVisible: false, //对话框
      form: {}, //保存点击以后当前学生s的信息
      key: null,
    };
  },
  created() {
    this.getStudentInfo();
  },
  methods: {
    getStudentInfo() {
      //分页查询所有试卷信息
      this.$axios(
        `/api/students/${this.pagination.current}/${this.pagination.size}`
      )
        .then((res) => {
          this.pagination = res.data.data;
        })
        .catch((error) => {});
    },
    //改变当前记录条数
    handleSizeChange(val) {
      this.pagination.size = val;
      this.getStudentInfo();
    },
    //改变当前页码，重新发送请求
    handleCurrentChange(val) {
      this.pagination.current = val;
      this.getStudentInfo();
    },
    checkGrade(studentId) {
      //修改学生信息
      this.dialogVisible = true;
      this.$axios(`/api/student/${studentId}`).then((res) => {
        this.form = res.data.data;
      });
    },
    deleteById(studentId) {
      //删除当前学生
      this.$confirm("确定删除当前学生吗？删除后无法恢复", "Warning", {
        confirmButtonText: "确定删除",
        cancelButtonText: "算了,留着吧",
        type: "danger",
      })
        .then(() => {
          //确认删除
          this.$axios({
            url: `/api/student/${studentId}`,
            method: "delete",
          }).then((res) => {
            this.getStudentInfo();
          });
        })
        .catch(() => {});
    },
    submit() {
      //提交更改
      this.dialogVisible = false;
      this.$axios({
        url: "/api/student",
        method: "put",
        data: {
          ...this.form,
        },
      }).then((res) => {
        console.log(res);
        if (res.data.code == 200) {
          this.$message({
            message: "更新成功",
            type: "success",
          });
        }
        this.getStudentInfo();
      });
    },
    handleClose(done) {
      //关闭提醒
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    search() {
      this.$axios("/api/students").then((res) => {
        if (res.data.code == 200) {
          let allStudent = res.data.data;
          let newPage = allStudent.filter((item) => {
            return item.studentName.includes(this.key);
          });
          this.pagination.records = newPage;
        } else {
          this.$message({
            type: "error",
            messages: "查询失败",
          });
        }
      });
    },
    resetPwd() {
      this.form.pwd = "123456";
      this.$axios({
        url: "/api/studentPWD",
        method: "put",
        data: {
          ...this.form,
        },
      }).then((res) => {
        console.log(res);
        if (res.data.code == 200) {
          this.$message({
            message: "已重置密码为123456",
            type: "success",
          });
        }
      });
    },
    downloadFile: function () {
      // 点击导出按钮
      let data = [{}];
      this.$axios(`/api/students`).then((res) => {
        let rs = res.data.data;
        let data = [{}];
        for (let k in rs[0]) {
          data[0][k] = k;
        }

        data = data.concat(rs);
        console.log(data);
        console.log(rs);

        this.downloadExl(data, "学生数据");
      });
    },
    downloadExl: function (json, downName, type) {
      // 导出到excel
      let keyMap = []; // 获取键
      for (let k in json[0]) {
        keyMap.push(k);
      }
      console.info("keyMap", keyMap, json);
      let tmpdata = []; // 用来保存转换好的json
      json
        .map((v, i) =>
          keyMap.map((k, j) =>
            Object.assign(
              {},
              {
                v: v[k],
                position:
                  (j > 25 ? this.getCharCol(j) : String.fromCharCode(65 + j)) +
                  (i + 1),
              }
            )
          )
        )
        .reduce((prev, next) => prev.concat(next))
        .forEach(function (v) {
          tmpdata[v.position] = {
            v: v.v,
          };
        });
      let outputPos = Object.keys(tmpdata); // 设置区域,比如表格从A1到D10
      let tmpWB = {
        SheetNames: ["mySheet"], // 保存的表标题
        Sheets: {
          mySheet: Object.assign(
            {},
            tmpdata, // 内容
            {
              "!ref": outputPos[0] + ":" + outputPos[outputPos.length - 1], // 设置填充区域
            }
          ),
        },
      };
      let tmpDown = new Blob(
        [
          this.s2ab(
            XLSX.write(
              tmpWB,
              {
                bookType: type === undefined ? "xlsx" : type,
                bookSST: false,
                type: "binary",
              } // 这里的数据是用来定义导出的格式类型
            )
          ),
        ],
        {
          type: "",
        }
      ); // 创建二进制对象写入转换好的字节流
      var href = URL.createObjectURL(tmpDown); // 创建对象超链接
      let a = document.createElement("a");
      a.href = href;
      a.download = downName + ".xlsx";
      a.click();
      // this.outFile.download = downName + ".xlsx"; // 下载名称
      // this.outFile.href = href; // 绑定a标签
      // this.outFile.click(); // 模拟点击实现下载
      setTimeout(function () {
        // 延时释放
        URL.revokeObjectURL(tmpDown); // 用URL.revokeObjectURL()来释放这个object URL
      }, 100);
    },
    s2ab: function (s) {
      // 字符串转字符流
      var buf = new ArrayBuffer(s.length);
      var view = new Uint8Array(buf);
      for (var i = 0; i !== s.length; ++i) {
        view[i] = s.charCodeAt(i) & 0xff;
      }
      return buf;
    },
  },
};
</script>
<style lang="less" scoped>
.all {
  padding: 0px 40px;
  .page {
    margin-top: 20px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .edit {
    margin-left: 20px;
  }
  .el-table tr {
    background-color: #dd5862 !important;
  }
}
.el-table .warning-row {
  background: #000 !important;
}

.el-table .success-row {
  background: #dd5862;
}
.searchBox {
  position: relative;
  padding: 10px 0px 10px 0px;
}
.search {
  margin-left: auto;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #eee;
  box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
  transition: border-color ease-in-out 0.15s, box-shadow ease-in-out 0.15s;
}
.search:hover {
  color: #0195ff;
  border-color: #0195ff;
}
</style>

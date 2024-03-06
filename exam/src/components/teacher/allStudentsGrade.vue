// 所有学生
<template>
  <div class="all">
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
      <el-table-column fixed="right" label="查看成绩" width="200">
        <template slot-scope="scope">
          <el-button
            @click="checkGrade(scope.row.studentId)"
            type="primary"
            size="small"
            >查看成绩</el-button
          >
          <el-button
            type="primary"
            size="small"
            @click="downloadFile(scope.row.studentId)"
            >导出表格</el-button
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
    ></el-pagination>
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
    };
  },
  created() {
    this.getAnswerInfo();
  },
  methods: {
    getAnswerInfo() {
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
      this.getAnswerInfo();
    },
    //改变当前页码，重新发送请求
    handleCurrentChange(val) {
      this.pagination.current = val;
      this.getAnswerInfo();
    },
    checkGrade(studentId) {
      this.$router.push({ path: "/grade", query: { studentId: studentId } });
    },
    downloadFile: function (studentId) {
      // 点击导出按钮
      let data = [{}];
      console.log(studentId);
      this.$axios(`/api/score/${studentId}`).then((res) => {
        let rs = res.data.data;
        let data = [{}];
        for (let k in rs[0]) {
          data[0][k] = k;
        }
        data = data.concat(rs);
        this.downloadExl(data, "同学考试成绩");
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
</style>

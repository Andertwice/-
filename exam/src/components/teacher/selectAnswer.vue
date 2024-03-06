//查询所有题库
<template>
  <div class="exam">
    <!-- <div class="searchBox">
      <select v-model="selected" class="search">
        <option value="null">请选择试题类型</option>
        <option value="multi">选择题</option>
        <option value="fill">填空题</option>
        <option value="judge">判断题</option>
      </select>
      <input
        type="text"
        placeholder="题目关键词"
        class="search"
        v-model="key"
      />
      <el-button type="primary" @click="search()">搜索试卷</el-button>
    </div> -->
    <div class="searchBox">
      <select v-model="type" class="search">
        <option value="null">请选择试题类型</option>
        <option value="multi">选择题</option>
        <option value="fill">填空题</option>
        <option value="judge">判断题</option>
      </select>
      <input type="text" placeholder="题目" class="search" v-model="key" />
      <el-button type="primary" @click="search()">搜索题目</el-button>
      <el-button type="primary" @click="downloadFile()">导出题目数据</el-button>
    </div>
    <el-table
      :data="pagination.records"
      border
      :row-class-name="tableRowClassName"
    >
      <el-table-column
        fixed="left"
        prop="subject"
        label="试卷名称"
        width="180"
      ></el-table-column>
      <el-table-column
        prop="question"
        label="题目信息"
        width="490"
      ></el-table-column>
      <el-table-column
        prop="section"
        label="所属章节"
        width="200"
      ></el-table-column>
      <el-table-column
        prop="type"
        label="题目类型"
        width="200"
      ></el-table-column>
      <el-table-column
        prop="score"
        label="试题分数"
        width="150"
      ></el-table-column>
      <el-table-column
        prop="level"
        label="难度等级"
        width="133"
      ></el-table-column>
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
        size: 4, //每页条数
      },
      key: null,
      type: null,
      optionValue: null,
    };
  },
  created() {
    this.getAnswerInfo();
  },
  mounted() {},
  methods: {
    getAnswerInfo() {
      //分页查询所有试卷信息
      this.$axios(
        `/api/answers/${this.pagination.current}/${this.pagination.size}`
      )
        .then((res) => {
          this.pagination = res.data.data;
          console.log(res);
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
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex % 2 == 0) {
        return "warning-row";
      } else {
        return "success-row";
      }
    },
    search() {
      this.$axios(`/api/answers/${this.type}`).then((res) => {
        if (res.data.code == 200) {
          console.log(res.data.data);
          let allAnswer = res.data.data;
          let newPage = allAnswer.filter((item) => {
            return item.question.includes(this.key);
          });
          this.pagination.records = newPage;
        }
      });
    },
    downloadFile: function () {
      // 点击导出按钮
      if (this.type == null) {
        this.$message({
          message: "请选择导出类型",
          type: "warning",
        });
      } else {
        this.$axios(`/api/answers/${this.type}`).then((res) => {
          let rs = res.data.data;
          console.log(rs);
          let data = [{}];
          for (let k in rs[0]) {
            data[0][k] = k;
          }
          data = data.concat(rs);
          if (this.type == "multi") {
            this.optionValue = "选择题";
          } else if (this.type == "judge") {
            this.optionValue = "判断题";
          } else if (this.type == "fill") {
            this.optionValue = "填空题";
          }
          this.downloadExl(data, this.optionValue + "题目");
        });
      }
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
.exam {
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

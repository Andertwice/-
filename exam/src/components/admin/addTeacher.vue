<!-- 添加教师 -->
<template>
  <div class="add">
    <el-tabs v-model="activeName">
      <el-tab-pane name="first">
        <span slot="label"><i class="el-icon-circle-plus"></i>添加老师</span>
        <section class="addBox">
          <el-form ref="form" :model="form" label-width="80px">
            <el-form-item label="教师姓名">
              <el-input v-model="form.teacherName"></el-input>
            </el-form-item>
            <el-form-item label="学院">
              <el-input v-model="form.institute"></el-input>
            </el-form-item>
            <el-form-item label="性别">
              <el-input v-model="form.sex"></el-input>
            </el-form-item>
            <el-form-item label="电话">
              <el-input v-model="form.tel"></el-input>
            </el-form-item>
            <el-form-item label="密码">
              <el-input v-model="form.pwd"></el-input>
            </el-form-item>
            <el-form-item label="身份证号">
              <el-input v-model="form.cardId"></el-input>
            </el-form-item>
            <el-form-item label="类型">
              <el-input v-model="form.type"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="onSubmit()">添加</el-button>
              <el-button type="text" @click="cancel()">删除</el-button>
            </el-form-item>
          </el-form>
        </section>
      </el-tab-pane>
      <el-tab-pane name="second">
        <span slot="label"><i class="el-icon-circle-plus"></i>批量导入</span>
        <div class="box">
          <input
            type="file"
            @change="importFile(this)"
            id="imFile"
            style="display: none"
            accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"
          />
          <a id="downlink"></a>
          <el-button type="primary" class="button" @click="downloadFile()"
            >点击下载规定格式范例</el-button
          >
          <el-button type="primary" class="button" @click="updateFile()"
            >点击开始导入</el-button
          >
          <span class="notice">注：请按照规定格式进行导入，谢谢</span>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
var XLSX = require("xlsx");
export default {
  data() {
    return {
      form: {
        //表单数据初始化
        teacherName: null,
        grade: null,
        major: null,
        clazz: null,
        institute: null,
        tel: null,
        email: null,
        pwd: null,
        cardId: null,
        sex: null,
        role: 2,
      },
      activeName: "first",
      imFile: null,
      outFile: null,
      errorMsg: "",
      excelData: null,
      fileName: "规定范式_教师.xlsx",
    };
  },
  mounted() {
    this.imFile = document.getElementById("imFile");
    this.outFile = document.getElementById("downlink");
  },
  methods: {
    onSubmit() {
      //数据提交
      this.$axios({
        url: "/api/teacher",
        method: "post",
        data: {
          ...this.form,
        },
      }).then((res) => {
        if (res.data.code == 200) {
          this.$message({
            message: "success",
            type: "success",
          });
        }
      });
    },
    cancel() {
      //取消按钮
      this.form = {};
    },
    updateFile: function () {
      // 点击导入按钮
      this.imFile.click();
    },
    // downloadFile: function (rs) {
    //   // 点击导出按钮
    //   let data = [{}];
    //   for (let k in rs[0]) {
    //     data[0][k] = k;
    //   }
    //   data = data.concat(rs);
    //   this.downloadExl(data, "菜单");
    // },
    downloadFile() {
      let a = document.createElement("a");
      a.href = "/static/xlsx/" + this.fileName;
      //路径中'/'为根目录，即index.html所在的目录
      a.download = this.fileName;
      a.click();
    },
    importFile: function () {
      // 导入excel
      this.fullscreenLoading = true;
      let obj = this.imFile;
      if (!obj.files) {
        this.fullscreenLoading = false;
        return;
      }
      var f = obj.files[0];
      var reader = new FileReader();
      let $t = this;
      reader.onload = function (e) {
        var data = e.target.result;
        if ($t.rABS) {
          $t.wb = XLSX.read(btoa(this.fixdata(data)), {
            // 手动转化
            type: "base64",
          });
        } else {
          $t.wb = XLSX.read(data, {
            type: "binary",
          });
        }
        let json = XLSX.utils.sheet_to_json($t.wb.Sheets[$t.wb.SheetNames[0]]);
        console.log(typeof json);
        $t.dealFile($t.analyzeData(json)); // analyzeData: 解析导入数据
      };
      if (this.rABS) {
        reader.readAsArrayBuffer(f);
      } else {
        reader.readAsBinaryString(f);
      }
    },
    analyzeData: function (data) {
      // 此处可以解析导入数据
      return data;
    },
    dealFile: function (data) {
      // 处理导入的数据
      console.log(data);
      this.imFile.value = "";
      this.fullscreenLoading = false;
      if (data.length <= 0) {
        this.errorDialog = true;
        this.errorMsg = "请导入正确信息";
      } else {
        this.excelData = data;
      }
      let i = 0;
      while (this.excelData[i] != null) {
        this.form = this.excelData[i];
        this.onSubmit();
        i++;
      }
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
    getCharCol: function (n) {
      // 将指定的自然数转换为26进制表示。映射关系：[0-25] -> [A-Z]。
      let s = "";
      let m = 0;
      while (n > 0) {
        m = (n % 26) + 1;
        s = String.fromCharCode(m + 64) + s;
        n = (n - m) / 26;
      }
      return s;
    },
    fixdata: function (data) {
      // 文件流转BinaryString
      var o = "";
      var l = 0;
      var w = 10240;
      for (; l < data.byteLength / w; ++l) {
        o += String.fromCharCode.apply(
          null,
          new Uint8Array(data.slice(l * w, l * w + w))
        );
      }
      o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)));
      return o;
    },
  },
};
</script>
<style lang="less" scoped>
.add {
  padding: 0px 40px;
  width: 80%;
}
.addBox {
  width: 400px;
}

.notice {
  color: rgba(255, 0, 0, 0.655);
  font-size: larger;
}
</style>

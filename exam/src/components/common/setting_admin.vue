<template>
  <div class="infoBox">
    <table style="padding: 0px 20px 10px 20px">
      <tr>
        <td>姓名：</td>
        <td>{{ this.form.adminName }}</td>
      </tr>
      <tr>
        <td>性别：</td>
        <td>{{ this.form.sex }}</td>
      </tr>
      <tr>
        <td>联系方式：</td>
        <td>{{ this.form.tel }}</td>
      </tr>
      <tr>
        <td>邮箱：</td>
        <td>{{ this.form.email }}</td>
      </tr>
    </table>
    <el-button type="primary" @click="updatePWD()" style="margin-left;: 30px">
      更改密码
    </el-button>
    <el-dialog title="请输入身份证号" :visible.sync="dialogVisible" width="30%">
      <section class="update">
        <el-form ref="form" label-width="80px">
          <el-form-item label="身份证号">
            <el-input v-model="cardId" type="password"></el-input>
          </el-form-item>
        </el-form>
      </section>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="verify()">提交</el-button>
      </span>
    </el-dialog>
    <el-dialog title="编辑试卷信息" :visible.sync="dialogVisible2" width="30%">
      <section class="update">
        <el-form ref="form" :model="form" label-width="80px">
          <el-form-item label="修改密码">
            <el-input v-model="newPWD" type="password"></el-input>
          </el-form-item>
          <el-form-item label="确认密码">
            <el-input v-model="confirmPWD" type="password"></el-input>
          </el-form-item>
        </el-form>
      </section>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submit()">提交</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      form: {},
      dialogVisible: false,
      dialogVisible2: false,
      newPWD: "",
      confirmPWD: "",
      cardId: "",
    };
  },
  created() {
    this.getInfo();
  },
  methods: {
    getInfo() {
      let Id = this.$route.params.id;
      console.log(Id);
      this.$axios(`/api/admin/${Id}`).then((res) => {
        this.form = res.data.data;
      });
    },
    updatePWD() {
      this.dialogVisible = true;
    },
    verify() {
      if (this.form.cardId == this.cardId) {
        this.dialogVisible = false;
        this.dialogVisible2 = true;
      }
    },
    submit() {
      //提交更改
      if (this.newPWD == this.confirmPWD) {
        this.dialogVisible = false;
        this.form.pwd = this.newPWD;
        this.$axios({
          url: "/api/admin",
          method: "put",
          data: {
            ...this.form,
          },
        }).then((res) => {
          console.log(res);
          if (res.data.code == 200) {
            this.$message({
              message: "success",
              type: "success",
            });
          }
          this.getInfo();
        });
      } else {
        this.$message({
          message: "两次密码不同",
          type: "warning",
        });
      }
    },
  },
};
</script>
<style>
td {
  padding: 0px 20px 10px 0px;
}
</style>

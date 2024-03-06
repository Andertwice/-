<!--学生建议页面-->
<template>
  <div>
    <span>{{ returnData }}</span>
    <el-button @click="getAdvise()">点击获取建议</el-button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      returnData: "",
      testDataString: "",
      id: this.$cookies.get("cid"),
      apiUrl:
        "https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/chat/completions_pro?access_token=24.e35e1102c59988e741691b5df497a6ce.2592000.1711973239.282335-54668196",
      Data: [
        {
          role: "user",
          content:
            "我将会给你几段数据，其中有选择题、填空题、判断题，这些数据是某个学生做错的题目，重复出现代表重复错误，我需要你分析数据并给出这名学生的后续学习建议，格式大致为：你当前错误选择题多少道，错误填空题多少道，错误判断题多少道，大部分错误集中在什么主题，然后给出后续学习建议",
        },
        {
          role: "assistant",
          content: "当然，我很乐意帮助您分析这些数据并提供改进建议",
        },
      ],
      testData: [{}],
      wrongData: [{}],
      Multi: [{}],
      Judge: [{}],
      Fill: [{}],
    };
  },
  created() {
    this.getWrongData();
    this.dealData();
  },
  methods: {
    //通过文心一言生成建议
    getAdvise() {
      this.$axios({
        url: this.apiUrl,
        method: "post",
        headers: {
          "Content-Type": "application/json",
        },
        data: {
          messages: [...this.Data],
        },
      }).then((res) => {
        console.log(res.data);
        this.returnData = res.data.result;
      });
    },
    //处理数据
    dealData() {
      console.log(this.Multi);
      console.log(this.Multi[0]);
      console.log(this.Multi[1]);
      // let string = JSON.stringify(this.Multi[0]);
      // console.log(string);

      // this.Data.push({
      //   role: "user",
      //   content:
      //     JSON.stringify(this.Multi) +
      //     JSON.stringify(this.Judge) +
      //     JSON.stringify(this.Fill),
      // });
      // console.log(this.Data);
    },
    //根据studentId获取错误题库
    getWrongData() {
      let Id = this.$cookies.get("cid");

      this.$axios(`/api/findErrorQuestionByStudentId/${Id}`).then((res) => {
        this.wrongData = res.data.data;
        this.getQuestionData();
      });
    },
    getQuestionData() {
      let i = 0;

      while (this.wrongData[i] != null) {
        if (this.wrongData[i].questionId <= 20000) {
          this.$axios(
            `/api/findBymultiId/${this.wrongData[i].questionId}`
          ).then((res) => {
            if (res.status == 200) this.Multi.unshift(res.data.data);
          });
        } else if (this.wrongData[i].questionId >= 30000) {
          this.$axios(`/api/findByfillId/${this.wrongData[i].questionId}`).then(
            (res) => {
              if (res.status == 200) this.Fill.unshift(res.data.data);
            }
          );
        } else {
          this.$axios(
            `/api/findByjudgeId/${this.wrongData[i].questionId}`
          ).then((res) => {
            if (res.status == 200) this.Fill.unshift(res.data.data);
          });
        }
        i++;
      }
    },
  },
};
</script>

<style scoped></style>

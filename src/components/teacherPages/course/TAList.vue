<template>
  <div>
    <div>
      <el-button class="btn" @click="writerTaID">
        <span>添加助教</span>
      </el-button>
    </div>
    <div>
      <el-tabs
        class="taTab"
        v-model="activeIndex"
        type="border-card"
        @tab-click="handleClick"
      >
        <el-tab-pane label="课程设置"> </el-tab-pane>
        <el-tab-pane label="教师团队"> </el-tab-pane>
        <el-tab-pane label="助教团队">
          <div>
            <el-table class="memberTab" :data="taList" height="480px"
                      :row-style="{ height: '50px' }"
                      :cell-style="{ padding: '0' }">
              <el-table-column prop="student_ID" label="学号" width="250px">
              </el-table-column>
              <el-table-column prop="name" label="姓名" width="250px">
              </el-table-column>
              <el-table-column prop="email" label="邮箱" width="400px">
              </el-table-column>
              <el-table-column width="120px">
                <template slot-scope="scope">
                  <el-button
                    type="text"
                    @click="open(scope.row)"
                  >删除
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
export default {
  name: "TAList",
  data() {
    return {
      activeIndex: "2",
      taList: [],
      dialogVisible: false,
      addTaID: "",
    };
  },
  methods: {
    handleClick(tab) {
      if (tab.index == 0) this.$router.push({ name: "info" });
      else if (tab.index == 1) this.$router.push({ name: "teachers" });
      else if (tab.index == 2) this.$router.push({ name: "tas" });
    },

    //删除助教的提示，确认后调用api删除助教
    open(row) {
      this.$confirm("此操作将从课程中删除该助教, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.deleteTa(row);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    //调用api向后端发送删除助教的id
    deleteTa(data) {
      let taInfo = new FormData()
      taInfo.append("student_ID",data.student_ID)
      taInfo.append("course_ID",this.$route.params.course_id)
      taInfo.append("is_student","0")
      this.$axios({
        url:"/take/deleteTakeCourse",
        method:"post",
        data:taInfo
      })
        .then(() => {
          this.taList.splice(data, 1);
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "error",
            message: "删除失败!请重试！",
          });
        });
    },

    //填写添加助教的id并检验
    writerTaID() {
      this.$prompt("请输入添加助教的学号", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        inputPattern: /^\d\d\d\d\d\d\d$/,
        inputErrorMessage: "学号格式不正确",
      })
        .then(({ value }) => {
          this.addTa(value);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "取消添加",
          });
        });
    },

    //调用api，向后端发送添加助教的id
    addTa(data) {
      let taInfo = new FormData()
      taInfo.append("student_ID",data)
      taInfo.append("course_ID",this.$route.params.course_id)
      taInfo.append("is_student","0")
      this.$axios({
        url:"/take/addTakeCourse",
        method:"post",
        data:taInfo
      })
        .then((response) => {
          if (response.data === 1) {
            this.loadDate();
            this.$message({
              type: "success",
              message: "添加成功！",
            });
          } else if (response.data === -1) {
            this.$message({
              type: "error",
              message: "添加失败！输入学生不存在！",
            });
          } else if (response.data === -3) {
            this.$message({
              type: "error",
              message: "添加失败！该学生已在课程中！",
            });
          }
        })
        .catch(() => {
          this.$message({
            type: "error",
            message: "添加失败！请重试!",
          });
        });
    },

    //加载助教列表数据
    loadDate() {
      this.$axios({
        url:"/take/getStudentInfoList",
        method:"get",
        params:{
          course_ID: this.$route.params.course_id,
          is_student: 0,
        }
      })
        .then((response) => {
          this.taList = response.data;
        })
        .catch();
    },
  },
  mounted() {
    this.loadDate();
  },
};
</script>

<style scoped>
.taTab {
  height: 100%;
  margin-top: 20px;
  margin-right: 50px;
  margin-left: 40px;
  background-color: white;
}

.btn {
  margin-left: 40px;
  margin-right: 20px;
}

.memberTab {
  width: 90%;
  margin-left: 60px;
  margin-top: 0;
}
</style>

<template>
  <div id="app">
<!--    <router-view></router-view>-->
    <router-view v-if="isRouterAlive" ></router-view>

    <el-button @click="hh" :style="{display:isShow}">老师课程主页</el-button>
    <el-button @click="studentCourseHome" :style="{display:isShow}">学生课程主页</el-button>

    <!--测试邮箱注册-->
    <el-form label-width="70px" style="margin-top: 50px">
      <el-form-item label="账号" prop="account_ID" style="width: 300px">
        <el-input v-model="account_ID"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email" style="width: 500px">
        <el-input style="width: 230px" v-model="email"></el-input>
        <el-button style="margin-left: 30px" @click="sendEmail">发送验证码</el-button>
      </el-form-item>
      <el-form-item label="密码" prop="password" style="width: 300px">
        <el-input v-model="password"></el-input>
      </el-form-item>
      <el-form-item label="验证码" prop="code" style="width: 300px">
        <el-input v-model="code"></el-input>
      </el-form-item>
    </el-form>
    <el-button style="margin-left: 150px" @click="register">注册</el-button>
  </div>
</template>

<script>
export default {
  name:"APP",
  provide() {
    return {
      reload: this.reload,
    }
  },
  data(){
    return{
      isRouterAlive: true,
      isShow: '',
      account_ID:'',
      email:'',
      password:'',
      code:'',
    }
  },
  methods:{
    reload() {
      this.isRouterAlive = false
      this.$nextTick(function() {
        this.isRouterAlive = true
      })
    },
    hh(){
      this.isShow='none'
      this.$router.push({
        name:'info',
        params:{
          course_id:'42024401'
        }
      })
    },

    studentCourseHome(){
      this.isShow='none'
      this.$router.push({
        name:'stuTasks',
        params:{
          course_id:'42024401'
        }
      })
    },

    sendEmail() {
      //没有判断邮箱格式是否正确
      let data = new FormData()
      data.append("account_ID",this.account_ID)
      data.append("email",this.email)
      if (this.email !== '') {
        this.$axios({
          url: "/account/sendEmail",
          method: "post",
          data: data
        }).then((response) => {
          if (response.data === 1) {
            this.$message({
              type: "success",
              message: "发送成功！",
            });
          } else if(response.data === -1) {
            this.$message({
              type: "error",
              message: "学号或工号已注册！请重试",
            });
          }
          else if(response.data === -2) {
            this.$message({
              type: "error",
              message: "学号或工号不存在！请重试",
            });
          }
          else{
            this.$message({
              type: "error",
              message: "发送失败！请重试",
            });
          }
        }).catch(() => {
          this.$message({
            type: "error",
            message: "发送失败！请重试",
          });
        })
      }
    },

    register(){
      //没有检验填写信息的正确性
      let data = new FormData();
      data.append("account_ID",this.account_ID)
      data.append("password",this.password)
      data.append("email",this.email)
      data.append("code",this.code)
      this.$axios({
        url:"/account/register",
        method:"post",
        data:data
      }).then((response)=>{
        if(response.data === 1) {
          this.$message({
            type: "success",
            message: "注册成功！",
          });
        }
        else{
          this.$message({
            type: "error",
            message: "填写信息错误！请重试",
          });
        }
      }).catch(()=>{
        this.$message({
          type: "error",
          message: "注册失败！请重试",
        });
      })
    }
  }
}
</script>

<style>
html,body,#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

body{
  margin: 0;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

</style>

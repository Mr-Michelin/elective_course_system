<template>
  <div id="register">
    <div class="title">用户注册 | REGISTER</div>
    <hr>
    <el-form
      :model="registerForm"
      :rules="rules"
      label-width="80px"
      :label-position="'left'"
      ref="registerForm"
    >
      <el-form-item prop="roles">
        <el-radio-group v-model="radio" >
          <el-radio :label="3" >学生</el-radio>
          <el-radio :label="6" >教师</el-radio>
          <el-radio :label="9" >管理员</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="账号" prop="account">
        <el-input v-model="registerForm.account"></el-input>
      </el-form-item>

      <el-form-item label="姓名" prop="name">
        <el-input v-model="registerForm.name"></el-input>
      </el-form-item>

      <el-form-item label="密码" prop="password">
        <el-input type="password" v-model="registerForm.password"></el-input>
      </el-form-item>

      <el-form-item label="重复密码" prop="password2" required>
        <el-input type="password" v-model="registerForm.password2"></el-input>
      </el-form-item>

      <el-row>
        <el-col :span="12">
          <el-form-item label="选择系别" prop="departmentName">
            <el-select style="width:95%" v-model="registerForm.departmentName" placeholder="请选择系别">
              <el-option v-for="item in departmentName" :key="item.id" :label="item.name" :value="item.name"></el-option>
            </el-select>
          </el-form-item>
        </el-col>

      </el-row>

      <el-form-item class="btn_box">
        <el-button type="primary" @click="registerClick">注 册</el-button>
        <el-button type="info" @click="$router.push('/login')">返回登录</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    var validatePassword2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.registerForm.password) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
    return {
      registerForm: {
        account: "",
        name: "",
        password: "",
        password2: "",
        departmentName: ""
      },
      rules: {
        account: [{ required: true, message: "请输入账号", trigger: "blur" }],
        name: [{ required: true, message: "请输入用户名", trigger: "blur" }],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }],
        password2: [
          { validator: validatePassword2, trigger: ["blur", "change"] }
        ],
        departmentName: [
          { required: true, message: "请选择系别", trigger: ["blur", "change"] }
        ]
      },
      departmentName: [],
    };
  },
  methods: {
    registerClick() {
      this.$refs.registerForm.validate(valid => {
        if (valid) {
          let obj = {
            account: this.registerForm.account,
            password: this.registerForm.password,
            name: this.registerForm.name,
            departmentName: this.registerForm.departmentName
          };
          console.log(obj);
          this.axios
            .post("/register", obj)
            .then(res => {
              if (res.data.code == 1) {
                this.$message("注册成功，请登录");
                this.$router.push("/login");
              } else {
                this.$message("该用户名已被占用，注册失败");
              }
            })
            .catch(() => {
              this.$message("无法连接服务器");
            });
        } else {
          return false;
        }
      });
    },
    getDepartment() {
      this.axios
        .get("/getDepartment")
        .then(res => {
          if (res.data.code == 1) {
            this.departmentName = res.data.data;
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("获取系别失败，服务器无法连接");
        });
    }
  },
  mounted() {
    this.getDepartment();
  }
};
</script>


<style lang="scss" scoped>
#register {
  .title {
    font-size: 1.5rem;
  }
  .btn_box {
    float: right;
  }
}
</style>

<template>
  <div id="addClass" v-loading.fullscreen.lock="loading" element-loading-text="系统正在自动分配合适的上课教室">
    <h1>添加上课时间</h1>
    <div class="container">
      <div class="main">
        <div class="left">
          <div class="title">
            <span class="step">STEP. 1</span>选择要添加上课时间的课程：
          </div>
          <div class="left_inner">
            <div v-if="courseData.length == 0" style="padding-left: 10px;">请添加课程</div>
            <div class="inner" v-for="item in courseData" :key="item.courseName">
              <div
                class="course"
                @click="chooseCourse = item.courseID"
                :class="{choose:(item.courseID == chooseCourse)}"
              >
                <div class="group">
                  <div class="name">{{item.courseName}}</div>
                </div>
                <div class="group">
                  <div class="departmentName">{{item.departmentName}}</div>
                </div>
                <div class="group">
                  <div class="credit">{{item.credit}}学分</div>
                  <div class="departmentName">{{item.departmentName | departmentName}}</div>
                </div>
              </div>
            </div>
          </div>
          <transition>
            <el-button
              type="success"
              icon="el-icon-check"
              circle
              v-if="chooseCourse!='' && chooseData.length != 0"
              @click="submit"
            ></el-button>
          </transition>
        </div>
        <div class="right">
          <div class="title">
            <span class="step">STEP. 2</span>请选择上课时间：
          </div>
          <div class="header">
            <div class="day">周一</div>
            <div class="day">周二</div>
            <div class="day">周三</div>
            <div class="day">周四</div>
            <div class="day">周五</div>
          </div>
          <div class="choose_card">
            <div class="class" v-for="(item,index) in data" :key="index">
              <div
                :class="{choose:item.choose}"
                class="class_inner"
                v-if="item.timeID == ''"
                @click="choose(item,index);"
              ></div>
              <div class="ban" v-if="item.timeID != ''"></div>
            </div>
          </div>
          <div class="tag">
            <i class="el-icon-info"></i> 请选择上方对应的上课时间，灰色区块为不可选择上课时间，提交课程后将不能更改。
          </div>
          <div class="choose_list">
            <div class="title">已选时间：</div>
            <div class="no_chooseData" v-if="chooseData.length == 0">请选课</div>
            <div class="list" v-for="(item,index) in chooseData" :key="index">
              <div>{{item.day | toWeek}}</div>
              <div>第{{item.time}}节</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      courseData: [],
      // 教师已排的课程
      data: [],
      // 选择的上课时间
      chooseData: [],
      chooseCourse: "",
      loading: false
    };
  },
  methods: {
    parseData(data) {
      let finalData = [];
      for (let i = 0; i < 55; i++) {
        finalData[i] = {
          timeID: "",
          courseID: "",
          classroomID: "",
          day: (i + 1) % 11 == 0 ? (i + 1) / 11 : parseInt((i + 1) / 11 + 1),
          time: (i + 1) % 11 == 0 ? 11 : (i + 1) %11,
          classroom: "",
          courseName: "",
          choose: false
        };
      }
      for (let i in data) {
        let day = parseInt(data[i].day);
        let time = parseInt(data[i].time);
        let index = (day - 1) * 11 + time-1;
        console.log(day,time,index)
        finalData[index] = data[i];
      }
      return finalData;
    },
    choose(item, index) {
      this.data[index].choose = !this.data[index].choose;
      if (this.data[index].choose) {
        let obj = {
          day: item.day.toString(),
          time: item.time.toString(),
          index: index
        };
        this.chooseData.push(obj);
      } else {
        for (let i in this.chooseData) {
          if (this.chooseData[i].index == index) {
            this.chooseData.splice(i, 1);
          }
        }
      }
    },
    submit() {
      this.$confirm("您确定要添加选择的上课时间吗?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.addClass();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "操作已取消"
          });
        });
    },
    addClass() {
      this.loading = true;
      let data = this.chooseData;
      for (let i in data) {
        delete data[i].index;
        data[i].courseID = this.chooseCourse;
      }

      for (let i in data) {
        this.axios
          .post("/addClass", data[i])
          .then(res => {
            if (res.data.code == 1) {
              if (i == data.length - 1) {
                setTimeout(() => {
                  this.loading = false;
                  this.$message.success("添加成功");
                  this.$router.push("/teacher");
                }, 2000);
              }
            }
          })
          .catch(err => {
            console.log(err);
            this.$message("服务器无法连接");
            this.loading = false;
          });
      }
    },
    getTeacherClass() {
      this.axios
        .get("/getTeacherClass?teacherID=" + this.$store.state.teacherID)
        .then(res => {
          if (res.data.code == 1) {
            this.data = this.parseData(res.data.data);
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("服务器无法连接");
        });
    },
    getTeacherCourse() {
      this.axios
        .get("/getTeacherCourse?teacherID=" + this.$store.state.teacherID)
        .then(res => {
          if (res.data.code == 1) {
            this.courseData = res.data.data;
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("服务器无法连接");
        });
    }
  },
  mounted() {
    this.getTeacherClass();
    this.getTeacherCourse();
  }
};
</script>

<style lang='scss' scoped>
.step {
  color: white;
  padding: 5px 10px;
  margin-right: 5px;
  background-color: #409eff;
  border-radius: 5px;
}
.choose {
  background-color: #67c23a;
  border: 2px solid #67c23a;
  color: white !important;
}
.main {
  width: 1250px;
  margin: 40px auto;
  display: flex;
  align-items: flex-start;
  .left {
    background-color: #fff;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
    width: 100%;
    height: auto;
    position: relative;
    .left_inner {
      display: flex;
      flex-wrap: wrap;
      margin: 0px -10px;
      margin-top: 20px;
      .inner {
        width: 25%;
        padding: 10px;
        box-sizing: border-box;
        .course {
          padding: 20px;
          border-radius: 10px;
          box-sizing: border-box;
          border: 3px solid rgba(0, 0, 0, 0.12);
          cursor: pointer;
          font-size: 14px;
          color: #606266;
          .group {
            display: flex;
            line-height: 1.7;
            .name {
              font-size: 20px;
              font-weight: bold;
            }
            .credit {
              margin-right: 10px;
            }
          }
        }
      }
    }
    .el-button {
      position: absolute;
      right: 20px;
      width: 50px;
      height: 50px;
      font-size: 1.2rem;
      bottom: -25px;
    }
  }
  .right {
    width: 430px;
    margin-left: 20px;
    background-color: #fff;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
    color: #606266;
    .title {
      margin-bottom: 20px;
      color: #303133;
    }
    .header {
      display: flex;
      .day {
        width: 20%;
        font-size: 14px;
        text-align: center;
      }
    }
    .choose_card {
      display: flex;
      flex-direction: column;
      flex-wrap: wrap;
      height: 640px;
      .class {
        width: 20%;
        height: 55px;
        padding: 10px;
        box-sizing: border-box;
        .class_inner {
          height: 100%;
          border-radius: 5px;
          border: 2px solid rgba(0, 0, 0, 0.2);
          cursor: pointer;
          &:hover {
            border: 2px solid #67c23a;
          }
        }
        .ban {
          height: 100%;
          border-radius: 5px;
          border: 2px solid rgba(0, 0, 0, 0.1);
          background: rgba(0, 0, 0, 0.2);
          cursor: not-allowed;
        }
      }
    }
    .tag {
      color: white;
      margin: 30px -20px;
      padding: 20px;
      background-color: #409eff;
      font-size: 14px;
      line-height: 1.7;
    }
    .list {
      background-color: #67c23a;
      color: white;
      display: flex;
      padding: 10px 20px;
      border-radius: 5px;
      justify-content: space-between;
      margin-bottom: 10px;
    }
  }
}
</style>

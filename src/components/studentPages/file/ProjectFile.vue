<template>
  <div id="CourseFile">
    <div class="FolderCard" v-for="info in project_files" v-if="show_folderName(info.fileName)">
      <div class="projectName" style="font-size: 18px;">【{{info.folderName}}】</div>
      <div v-for="file in info.fileName" class="fileName" @click="handleDownload(info.folderName,file)">
        {{file}}
      </div>
      <el-divider></el-divider>
    </div>
  </div>
</template>

<script>
export default {
  name: "ProjectFile",
  props:{
    course_id:{
      type: String,
      default: '',
    },
  },
  data(){
    return{
      project_files: [],
    }
  },
  mounted() {
    var id=this.course_id;
    this.$axios.get('/file/getDirectoryFiles',{
      params:{
        course_ID:id,
        isProject:1,
      }
    }).then((response)=>{
      this.project_files=response.data;
    })
  },
  methods: {
    show_folderName(fileName_list){
      if(fileName_list.length>0){
        return true;
      }
      else{
        return false;
      }
    },
    handleChange(val) {
      console.log(val);
    },
    handleDownload(folderName,fileName) {
      var id=this.course_id;
      var path='/实验资料/'+folderName;
      let data = new FormData();
      data.append("course_ID", id);
      data.append("path", path);
      data.append("file_name", fileName);
      this.downloadFile(data, fileName);
    },
    downloadFile(data) {
      this.$axios({
        url: "/file/downloadFile",
        method: "post",
        data: data,
        responseType: "blob",
      }).then((response) => {
        console.log(response);
        let blob = new Blob([response.data]);
        console.log(blob);
        const disposition = response.headers["content-disposition"];
        //获得文件名
        let fileName = disposition.substring(
            disposition.indexOf("filename=") + 9,
            disposition.length
        );
        //解码
        fileName = decodeURI(fileName);
        if (window.navigator.msSaveOrOpenBlob) {
          navigator.msSaveBlob(blob, fileName);
        } else {
          const link = document.createElement("a");
          link.href = window.URL.createObjectURL(blob);
          link.download = fileName;
          link.click();
          //释放内存
          window.URL.revokeObjectURL(link.href);
        }
      });
    },
  }
}
</script>

<style scoped>
.FolderCard{
  margin-top: 30px;
  margin-left: 50px;
  margin-right: 50px;
}
.fileName{
  margin-top: 15px;
  margin-left: 30px;
  color: rgba(11,159,250,1);
  cursor: context-menu;
}
.fileName:hover{
  color: rgba(0,0,250,0.6);
}
a:hover{
  text-decoration: none;
}
</style>
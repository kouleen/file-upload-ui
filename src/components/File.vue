<template>
  <div class="hello">
      <el-form :inline="true" :model="queryFile" class="demo-form-inline">
        <el-form-item >
            <el-input v-model="queryFile.name" placeholder="文件名称"></el-input>
        </el-form-item>
        <el-form-item>
            <el-button type="primary" @click="onSubmit">查询</el-button>
        </el-form-item>

      </el-form>

      <el-upload
        class="upload-demo"
        ref="upload"
        action="http://localhost/file/upload"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        :file-list="fileList"
         multiple
      
        :auto-upload="false">
        <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
        <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
      </el-upload>

      <el-table
        :data="tableData" height="800" border
        style="width: 100%">
      <el-table-column
        prop="name" label="文件名称" width="300">
      </el-table-column>
      <el-table-column
        prop="size" label="文件大小" width="120">
      </el-table-column>
      <el-table-column
        prop="click" label="文件热度" width="80">
      </el-table-column>
      <el-table-column
        prop="path" label="文件地址" width="800">
      </el-table-column>
      <el-table-column
        prop="createTime" label="创建时间" >
      </el-table-column>
      <el-table-column
        fixed="right" label="操作" width="100">
      <template slot-scope="scope">
        <el-button :plain="true" @click="handleClick(scope.row.id)" type="text" size="small">下载</el-button>
      </template>
    </el-table-column>
    </el-table>
  <div class="block">
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[10, 20, 30, 50,100,200,400]"
      :page-size=pageSize
      layout="total, sizes, prev, pager, next, jumper"
      :total=total>
    </el-pagination>
  </div>
  </div>
</template>

<script>
const baseUrl = 'http://localhost:8080'
export default {
  // <!-- layout="total, sizes, prev, pager, next, jumper" -->
  name: 'File',
  data(){
      return{
          tableData: [],
          queryFile: {
            name: '',
            page: 1,
            limit: 10
          },
          fileList: [
          ],
          currentPage: 20,
          total: 10,
          pageSize: 5,
      }
  },
  computed: {

  },
    methods: {
      handleRemove(file, fileList) {
        console.log(file)
      },
      handlePreview(file) {
        console.log("你要上传的文件" + file);
      },
      submitUpload() {
        this.$refs.upload.submit();
      },
      handleSizeChange(val) {
        this.queryFile.limit = val
        this.queryFile.page = 1 
        this.gerQueryFile()
      },
      handleCurrentChange(val) {
        this.queryFile.limit = this.pageSize
        this.queryFile.page = val
        this.gerQueryFile()
      },
      gerQueryFile(){
        let _this = this
        this.$http({
            method: 'get',
            url: 'http://localhost/file/queryFile',
            params: {
                page: _this.queryFile.page,
                limit: _this.queryFile.limit,
                name: _this.queryFile.name
            }
        }).then(function(res){
            if(res.data.code == 200){
              _this.tableData = res.data.data.records
              _this.tableData.forEach(element => {
                element['size'] = _this.getFileSize(element.size)
              });
              // 当前页
              _this.currentPage = res.data.data.current
              // 总条数
              _this.total = res.data.data.total
              // 每页数量
              _this.pageSize = res.data.data.size
            }
        }).catch(function(error){
          console.log(error)
        })
      },
      getFileSize(fileSize) {
        let i = 0
        while (fileSize > 1024) {
            fileSize = fileSize / 1024;
            i++;
        }
        let unit = "";
        switch (i) {
            case 0:
                unit = "B";
                break;
            case 1:
                unit = "KB";
                break;
            case 2:
                unit = "MB";
                break;
            case 3:
                unit = "GB";
                break;
            case 4:
                unit = "TB";
                break;
            case 5:
                unit = "PB";
                break;
            default:
                unit = "XB";
        }
        fileSize = fileSize.toFixed(2)
        return fileSize + unit
      },
      onSubmit() {
        this.gerQueryFile()
      },
      handleClick(row) {
        this.getDataByUrl(row)
      },
      getDataByUrl(row){
        let _this = this;
        this.$http({
            method: 'get',
            url: 'http://localhost/file/query/' + row,
        }).then(function(res){
            if(res.data.code == 200){
                _this.$message({
                  message: res.data.msg,
                  type: 'success'
                });
                window.location.href = res.data.data.path
                _this.gerQueryFile()
            }else{
              _this.$message({
                message: res.data.data,
                type: 'error'
              });
            }
        }).catch(function(error){
          console.log(error)
        })
      }
  },
  created: function(){
    this.gerQueryFile()
  }
}
</script>

<style>
  body {
    margin: 0;
  }
</style>



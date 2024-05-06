<template>
    <div id="Excel">
      <!-- 上传 -->
      <el-upload
        action
        accept=".xlsx, .xls"
        :auto-upload="false"
        :show-file-list="false"
        :on-change="handle"
      >
        <el-button type="primary" class="imExcelBtn">导入Excel</el-button>
      </el-upload>
  
      <!-- 表格名称及导出和关闭预览的按钮 -->
      <div class="excelName" v-if="tableKey">
        <div class="left"></div>
        <div class="center">{{ excelName }}</div>
        <div class="right">
          <el-button type="success" @click="exportExcel">导出为Excel</el-button>
          <!-- <el-button type="danger" @click="closeExcel">关闭预览</el-button> -->
        </div>
      </div>
      <!-- 表格 -->
      <!-- :key="Math.random()"解决修改数据不刷新的问题 -->
      <el-table
        :data="tableData"
        style="width: 100%"
        max-height="500"
        :key="Math.random()"
        :header-cell-style="{
          background: '#d2d28e',
          color: '#000000',
        }"
        v-if="tableKey"
        id="excelTable"
        border
        stripe
      >
      <!-- 第一个el-table-column 可根据需求删除-->
        <!-- <el-table-column
          header-align="center"
          align="center"
          type="index"
          label="序号"
          width="50"
        ></el-table-column> -->
        <!-- 表格数据 -->
        <el-table-column
          v-for="(item, index) in tableHeader"
          :key="index"
          :prop="item"
          :label="item"
          width="width">
        </el-table-column>
        <!-- 操作表格数据 -->
        <el-table-column
          fixed="right"
          label="操作"
          class="editColumn"
         >
          <!-- {row,$index}当前项数据和下标位置 -->
          <template slot-scope="{row,$index}">
            <el-button @click="openDialogBtn(row,$index)" type="primary" size="small">修改数据</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 修改数据对话框 -->
      <el-dialog
        title=""
        :visible.sync="dialogVisible"
        width="width"
        :before-close="dialogBeforeClose">
        <el-form ref="form" :model="rowData" label-width="80px">
          <el-form-item :label="item" v-for="(item,index) in tableHeader" :key="index">
            <!-- rowData[item]：取对象值 -->
            <el-input v-model="rowData[item]"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editBtn">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </template>
 <script>
 import * as XLSX from "xlsx/xlsx.mjs";
import FileSaver from "file-saver";
export default {
  name: "ExcelSheet",
  data() {
    return {
      tableKey: false, //显示与隐藏
      excelName: "", //excel表格名称
      tableData: [], //表格数据
      tableHeader: [], //表格表头
      dialogVisible: false, //对话框开关
      rowData: {}, //修改后的数据
      tableIndex: 0, //修改数据的下标值
    };
  },
  created() {
    if (localStorage.getItem("tableData")) {
      this.tableData = JSON.parse(localStorage.getItem("tableData"));
      console.log("mmmm",this.tableData)
    }
  },
  methods: {
    // 打开对话框
    openDialogBtn(row, index) {
      this.dialogVisible = true;
      this.tableIndex = index;
      let emptyObj = {};
      // 遍历表头创建对象，键名：表头，键值：空
      this.tableHeader.forEach((item) => {
        emptyObj[item] = "";
      });
      // 与拿到的对象合并
      this.rowData = { ...emptyObj, ...row };
    },
    // 修改对话框数据
    editBtn() {
      // 拿到未修改对应位置数据
      let indexData = this.tableData[this.tableIndex];
      // 将原数据与修改后的新数据进行对象合并
      this.tableData[this.tableIndex] = { ...indexData, ...this.rowData };
      this.dialogVisible = false;
      console.log(this.tableData);
    },
    // 关闭对话框
    dialogBeforeClose() {
      this.dialogVisible = false;
    },
    readFile(file) {
      //文件读取
      return new Promise((resolve) => {
        let reader = new FileReader();
        reader.readAsBinaryString(file); //以二进制的方式读取
        reader.onload = (ev) => {
          resolve(ev.target.result);
        };
      });
    },
    // 上传文件状态改变时的钩子，添加文件、上传成功和上传失败时都会被调用
    async handle(ev) {
      //改变表格key值
      this.tableKey = true;
      let file = ev.raw;
      console.log("上传的文件", file);
      this.excelName = file.name;
      //截取表格文件名
      this.excelName = this.excelName.substring(
        0,
        this.excelName.lastIndexOf(".")
      );
      console.log("上传的未解析源文件", file);
      if (!file) {
        console.log("文件打开失败");
        return;
      } else {
        let data = await this.readFile(file);
        let workbook = XLSX.read(data, { type: "binary" }); //解析二进制格式数据
        console.log("二进制数据的解析：", workbook);
        let worksheet = workbook.Sheets[workbook.SheetNames[0]]; //获取第一个Sheet
        // 调用解析表头方法
        this.getHeader(worksheet);
        let result = XLSX.utils.sheet_to_json(worksheet); //转换为json数据格式
        console.log("最终解析的 json 格式数据：", result);
        // 表格数据
        this.tableData = result;
        // 进行本地存储
        localStorage.setItem("tableData", JSON.stringify(result));
      }
    },
    // 解析出表格表头
    getHeader(sheet) {
      const headers = [];
      const range = XLSX.utils.decode_range(sheet["!ref"]); // worksheet['!ref'] 是工作表的有效范围
      let C;
      /* 获取单元格值 start in the first row */
      const R = range.s.r; // 行 // C 列
      let i = 0;
      // s:开始start，e:结束end
      for (C = range.s.c; C <= range.e.c; ++C) {
        var cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })]; /* 根据地址得到单元格的值find the cell in the first row */
        var hdr = "UNKNOWN" + C; // 如果有空表头，会替换为您想要的默认值replace with your desired default
        // XLSX.utils.format_cell 生成单元格文本值
        if (cell && cell.t) hdr = XLSX.utils.format_cell(cell);
        if (hdr.indexOf("UNKNOWN") > -1) {
          if (!i) {
            hdr = "__EMPTY";
          } else {
            hdr = "__EMPTY_" + i;
          }
          i++;
        }
        headers.push(hdr);
      }
      // 保存至data中
      this.tableHeader = headers;
      return headers;
    },
    // 关闭表格预览
    closeExcel() {
      this.tableKey = false;
      //表格数据置空
      this.tableData = [];
      this.tableColumn = [];
    },
    //导出表格为Excel
    exportExcel() {
      /* generate workbook object from table */
      let xlsxParam = { raw: true }; // 导出的内容只做解析，不进行格式转换
      let table = document.querySelector("#excelTable").cloneNode(true);
      // 移除固定列：解决导出两张表的问题
      table.removeChild(table.querySelector(".el-table__fixed-right"));
      // 导出的时候去除操作一
      //****************************************************
      let wb = XLSX.utils.table_to_book(table, xlsxParam);
      /* get binary string as output */
      // 1. 导出 Excel 对象
      let wbout = XLSX.write(wb, { bookType: "xlsx", bookSST: true, type: "array" });
      try {
        // 2. 将 Excel 对象转换为 Blob 对象，并指定文件的类型为 'application/octet-stream'
        FileSaver.saveAs(new Blob([wbout], { type: "application/octet-stream" }), "fileName.xlsx");
      } catch (e) {
        // 3. 如果出现异常，打印错误信息和 wbout
        if (typeof console !== "undefined") {
          console.log(e, wbout);
        }
      }
      // 4. 返回 wbout
      return wbout;
    },
  },
};

</script>
  
  <style lang='less' scoped>
  .excelName {
    display: flex;
    justify-content: space-between;
    font-size: 30px;
    margin-bottom: 20px;
  }
  .imExcelBtn {
    margin: 10px 0;
  }
  </style>
<template>
<div class="commonPlane">
    <div class="filterPanel" > 
    </div>
    <div class='tablezoom'>
        <div class="tablePanel">
            <el-table
                ref="multipleTable"
                :data="pagination.resultList"
                @select="onSelect"
                @select-all="onSelectALL"
                style="width: 100%">            <!--  :selectable="selectable" -->
                <el-table-column type="selection"  :reserve-selection="true"></el-table-column>
                <el-table-column label="日期" width="180">
                    <template slot-scope="scope">
                        <i class="el-icon-time"></i>
                        <span style="margin-left: 10px">{{ scope.row.date }}</span>
                    </template>
                </el-table-column>
                <el-table-column label="姓名" width="180">
                    <template slot-scope="scope">
                        <el-popover trigger="hover" placement="top">
                        <p>姓名: {{ scope.row.name }}</p>
                        <p>住址: {{ scope.row.address }}</p>
                        <div slot="reference" class="name-wrapper">
                            <el-tag size="medium">{{ scope.row.name }}</el-tag>
                        </div>
                        </el-popover>
                    </template>
                </el-table-column>
                <el-table-column
                    prop="address"
                    label="地址"
                    :formatter="formatter">
                </el-table-column>
                <el-table-column
                    prop="tag"
                    label="标签"
                    width="100"
                    :filters="[{ text: '同学', value: '同学' }, { text: '老师', value: '老师' }]"
                    :filter-method="filterTag"
                    filter-placement="bottom-end">
                    <template slot-scope="scope">
                        <el-tag
                        :type="scope.row.tag === '家' ? 'primary' : 'success'"
                        disable-transitions>{{scope.row.tag}}</el-tag>
                    </template>
                    </el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button
                        size="mini"
                        @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button
                        size="mini"
                        type="danger"
                        @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="pagination.currentPage"
                :page-sizes="pagination.pageSizes"
                :page-size="pagination.pageSize"
                :layout="pagination.layout"
                :total="pagination.total">
            </el-pagination>

        </div>
        <div class='footPanel'>
            <el-row>
                <el-col >
                     <el-button type='primary' @click="cancle">取消</el-button>
                    <el-button type='primary' @click="confirm">确认</el-button>
                </el-col>
            </el-row>
        </div>
</div>
</template>
<script>
import data from '@/assets/js/tableData.js'
import pagination from '@/assets/js/pagination.js'

export default {

  mixins: [data,pagination],
  data () {
    return {
      selectable: '',
      selected:[],// 实时选中数据，用于回显 记录需要回显的数据 实时更新
      falseList:[],// 记录取消的数据
      trueList:[],// 记录选中的数据
      resultList:[],
      pagination:{},
    }
  },
  created:function() {
      debugger
      var echoInfo =  window.opener.document.getElementById("parentValue").value;
      
      for(var i=0;i<echoInfo.split(',').length;i++)
      {
         var element = {id:'',name:''};
         element.id = echoInfo.split(',')[i];
          element.name = echoInfo.split(',')[i];
          this.selected.push(element);
      }
      
      this.init();
    //   this.echoSelectedInfo();
  },
  updated:function(){
    debugger
    if (this.selected !=undefined && this.selected !='') {
  
        //   for(var j=0;j<this.selected.length;j++)
        //   {
        //     if(this.pagination.resultList.indexOf(this.selected[j]) !== -1)
        //     {
        //         this.$refs.multipleTable.toggleRowSelection(this.selected[j]);
        //     }
        //   }  
        for(var i=0;i<this.pagination.resultList.length;i++)
        {
          for(var j=0;j<this.selected.length;j++)
          {

            if(this.pagination.resultList[i].name===this.selected[j].name)
            {
                this.$refs.multipleTable.toggleRowSelection(this.pagination.resultList[i]);
            }
          }  
        }
       
    } else 
    {
          this.$refs.multipleTable.clearSelection();
    }
      
  },
  watch:{
      pagination:function(op)
      {
          debugger;
          this.resultList = op.resultList ;
      },
      falseList:function(value)
      {
          
          if(this.trueList.length == 0)
          {
              var totalArray = this.mergeArrayRelaseSame(this.selected,this.trueList);
              if(this.falseList !='' && this.falseList.length !=0)
              {
                  // 总数据中删除 取消的数据
                  
                  var len = this.falseList.length ;
                  for(var i=0;i<len;i++)
                  {
                      var totalLen = totalArray.length ;
                      for(var j=totalLen -1 ;j>=0;j--)
                      {
                        
                        if(totalArray[j].id == this.falseList[i].id){
                            totalArray.splice(j,1);
                            break
                        }
                      }
                  }
              }
              this.selected = totalArray ;
              console.log(this.selected);
          }
           
      },
      trueList:function(value)
      {
          // 开始回显的数据+进入页面选中的数据-清除选中的数据
          // selected+trueList-falseList
          var totalArray = this.mergeArrayRelaseSame(this.selected,this.trueList);
          if(this.falseList !='' && this.falseList.length !=0)
          {
             // 总数据中删除 取消的数据
            var totalLen = totalArray.length ;
            var len = this.falseList.length ;
            for(var i=0;i<len;i++)
            {
                for(var j=totalLen -1 ;j>=0;j--)
                {
               
                if(totalArray[j].id == this.falseList[i].id) // 判断数据是否需要删除的依据
                { 
                    totalArray.splice(j,1);
                    break
                }
                }
            }
          }
          this.selected = totalArray ;
          console.log(this.selected);
           
      },
  },
  mounted() {
   
//      var that = this ;
//      window.addEventListener('message', function(e){

//         if(e.data !=undefined && e.data !='' ){
//             debugger
//             if(e.data ==undefined || e.data ==''){
//                 return ;
//             } 
//             //  var json = JSON.parse(e.data);
//             var json =e.data ;
//             if(json.type =="echo")
//             {
//                 // that.initParent(json.data);
//                   that.selected = json.data ;
//                   that.echoSelectedInfo();
//             }
//         }

//     });
  },
  methods:
  {
    // echoSelectedInfo:function () {
    // if (this.selected !=undefined && this.selected !='') {
  
    //     for(var i=0;i<this.pagination.resultList.length;i++)
    //     {
    //       for(var j=0;j<this.selected.length;j++)
    //       {

    //         if(this.pagination.resultList[i].name===this.selected[j].name)
    //         {
    //             this.$refs.multipleTable.toggleRowSelection(this.selected[j]);
    //         }
    //       }  
    //     }
    // } else 
    // {
    //       this.$refs.multipleTable.clearSelection();
    // }
    // },
    confirm:function()
    {
        debugger;
    
        // type 用来
        var json = {type:'echoInfo',data:[]};
        json.data = this.selected ;
        // var jsonStr = JSON.stringify(json);
        // 使用 h5中的postMessage 能解决页面跨域问题 第一个参数表示 需要回传的数据，第二个参数时url * 表示不限制
        window.opener.postMessage(json,'*');
        window.close();
    },
    cancle:function()
    {
        // 关闭子页面弹框
        window.close();
    },
    initParent:function(op)
    {
        debugger
        if(op !=undefined && op !='')
        {
           that.selected =op ;
        }
        
    },
    init: function () {
        debugger
        this.pagination.resultList = this.tableData ;
        this.resultList = this.tableData ;
    },
    
    //表格当前页全选出发函数
    onSelectALL:function(rows)
    {
        this.indeterminate = true ;
        var lenNow = this.resultList.length;
        // 判断勾选全选本页是选中 还是取消
        if(rows.indexOf(this.resultList[0]) !== -1)
        {
            debugger
            //选中
            for(var i=0; i<lenNow ;i++)
            {
                // 记录撤销数组 选中后删除数据操作
                for(var n = 0;n<this.falseList.length;n++)
                {
                    if(this.falseList[n].id== this.resultList[i].id)
                    {
                        this.falseList.splice(n,1)
                    }
                }
                // 记录选中数据数据 选中部分操作
                if(this.trueList.length !== 0)
                {
                    if(this.trueList.indexOf(this.resultList[i]) === -1)
                    {
                        this.trueList.push(this.resultList[i]);
                    }
                }
                else
                {
                    if(this.trueList.indexOf(this.resultList[i])=== -1)
                    {
                       this.trueList.push(this.resultList[i]);
                    }
                }
            }
        }
        else
        {
            debugger
            // 取消操作
            for(var j=0;j<lenNow;j++)
            {
                // 记录撤销的操作 往记录撤销数组中添加数据
                if(this.falseList.length !== 0)
                {
                    if(this.falseList.indexOf(this.resultList[i])=== -1)
                    {
                        this.falseList.push(this.resultList[j]);
                    }
                }
                if(this.falseList.length === 0)
                {
                    this.falseList.push(this.resultList[j]);
                }
                // 记录选中的相关操作 撤销时，删除已选中的数组中的取消的数据的删除
                if(this.trueList.length !==0)
                {
                    for(var n = 0; n<this.trueList.length;n++)
                    {
                        if(this.trueList[n].id === this.resultList[j].id)
                        {
                            this.trueList.splice(n,1);
                        }
                    }
                }
            }
        }
        
    },
    // 选中数据操作
    onSelect:function(rows,row)
    {
        this.indeterminate = true ;
        var selected = rows.length && rows.indexOf(row) !== -1
        var lenFalse = this.falseList.length
        var lenTrue = this.trueList.length
        if(selected)
        {
            // 选中
            // 对记录撤销选中的计算 falseList
            if(lenFalse !== 0)
            {
                for(var i=0;i<lenFalse;i++)
                {
                    if(this.falseList[i].id === row.id)
                    {
                        this.falseList.splice(i,1);
                        break
                    }
                }
            }
            // 对记录选中的数据的计算 trueList
            if( lenTrue !=0)
            {
                if(this.trueList.indexOf(row.id) === -1)
                {
                    this.trueList.push(row);
                }
            }
            if(lenTrue ==0)
            {
                this.trueList.push(row);
            }

        }
        else
        {
            // 取消 falseList
            this.falseList.push(row);
            // trueList 操作
            for(var i=0;i<lenTrue;i++)
            {
                if(this.trueList[i].id===row.id)
                {
                    this.trueList.splice(i,1);
                    break
                }
            }

        }
    },
    // 数据合并并去重  arr [{id:'',value:''},...]
    mergeArrayRelaseSame: function(arr1,arr2)
    {
        var mergeArray = arr1.concat(arr2);
        var newArray = [] ; //存放去重后的数据的新数组
        for(var i=0;i<mergeArray.length;i++)
        {
            var item1 = mergeArray[i];
            var flag = true ;
            for(var j=0;j<newArray.length;j++)
            {
                var item2 = newArray[j];
                if(item1.id == item2.id)
                {
                   flag = false ;
                }
            }
            if(flag)
            {
                newArray.push(item1);
            }
        }
        return newArray ;

    },
    filterTag: function(value, row) {
        return row.tag === value;
    },
    handleEdit: function(index,row)
    {

    },
    handleDelete: function(index,row)
    {

    },
    formatter:function(row, column){
        return row.address;
    }
    
    
  }
}
</script>
<style scoped>
 
 .tablezoom .tablePanel
 {
     width: auto;
     min-height: 400px;
     max-height: 600px;
     overflow-y: auto;
     
 }
 .footPanel{
     float: right;
 }
</style>

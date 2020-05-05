<!-- 页面回显 -->
<template>
    <div class='mainPlane'>
        <el-row>
             <el-col :span='6'>
                 <el-tooltip class="item" effect="dark" :content="echo" placement="top-start">
                    <el-input id="parentValue" v-model="echo" placeholder="能确定回显内容的唯一值,如id" disabled ></el-input>
                 </el-tooltip>
            </el-col>
            <el-col :span='3'>
                <!-- <router-link to='/elementTable'>跳转页面</router-link> -->
                <el-button type="primary" size="small" @click="openTablePage">点击弹出数据选择页面</el-button>
            </el-col>
        </el-row>        
    </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
//import引入的组件需要注入到对象中才能使用
components: {

},
data() {
    //这里存放数据
    return {
        echoData:[],
        echo:'',
    };
},
//监听属性 类似于data概念
computed: {
},
//监控data中的数据变化
watch: {
   
},
//方法集合
methods: {

    init:function()
    {
        debugger
        if(this.echoData !=undefined && this.echoData != '')
        {
            var arr = JSON.parse(this.echoData);
            var name ="";
            for(var i=0;i<arr.length;i++)
            {
                name += ","+arr[i].name ;
            }
            this.echo = name.substring(1);
        }
    },
    openTablePage: function()
    {
        debugger;
        var BaseUrl ="" ;
        var loc = window.location;
        if (loc.origin){
            BaseUrl = loc.origin;
        }
        else
        {
            BaseUrl = loc.protocol+"//"+loc.host;
        }
        var json = {type:'echo',data:[]};
        if(this.echoData !=undefined && this.echoData != '')
        {
               
            json.data =this.echoData;
               
               
        }
        var childWindow = window.open(BaseUrl+"/#/elementTable?data="+json, 'newwindow', 'height=800, width=1200, top=0, left=0, toolbar=no, menubar=no, scrollbars=no, resizable=no, location=no, status=no');
         // 父页面传递数据到子页面，子页面回显
        
        // var that = this ;
        // childWindow.onload = function()
        // {
        //     if(that.echoData !=undefined && that.echoData != '')
        //     {
        //         var json = {type:'echo',data:[]};
        //         json.data =that.echoData;
               
        //         childWindow.postMessage(json,'*');
        //     }
           
            
        // }
       
        
        
    },
    //子页面关闭后更新数据
    receiveMessage:function(jsonStr)
    {
       var name ="";
       for(var i= 0;i<jsonStr.length;i++)
       {
           if(jsonStr[i].name !="")
           {
              name += ","+jsonStr[i].name ;
           }    
       }
       this.echo = name.substring(1);
    },

},
//生命周期 - 创建完成（可以访问当前this实例）
created() {
    
    this.init();
},
//生命周期 - 挂载完成（可以访问DOM元素）
mounted() {
    var that = this ;
     window.addEventListener('message', function(e){

        if(e.data !=undefined && e.data !='' ){
          debugger
          if(e.data ==undefined || e.data ==''){
                return ;
            } 
            // var json = JSON.parse(e.data);e.data;
            var json = e.data;
            if(json.type =="echoInfo")
            {
                that.echoData = json.data ;
                that.receiveMessage(json.data);
            }
        }

    });

},
beforeCreate() {}, //生命周期 - 创建之前
beforeMount() {}, //生命周期 - 挂载之前
beforeUpdate() {}, //生命周期 - 更新之前
updated() {}, //生命周期 - 更新之后
beforeDestroy() {}, //生命周期 - 销毁之前
destroyed() {}, //生命周期 - 销毁完成
activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>

<style scoped>

</style>
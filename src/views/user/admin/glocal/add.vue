<template>
  <section class="cms-body" v-loading="loading">
    <!-- 返回组件 -->
    <cms-back></cms-back>        
    <el-form  ref="form" :model="dataInfo" :rules="rules" 
        class="cms-form" label-width="162px">
        <el-form-item label="用户名" class="flex-50"  prop="username">
               <el-input v-model="dataInfo.username" class="cms-width"></el-input>
        </el-form-item>        
        <el-form-item label="电子邮箱" class="flex-50" prop="email">
               <el-input v-model="dataInfo.email" class="cms-width"></el-input>
        </el-form-item>
        <el-form-item label="密码" class="flex-50" prop="password">
                <el-input v-model="dataInfo.password" class="cms-width" type="password"></el-input>                              
            </el-form-item>
            <el-form-item label="确认密码" class="flex-50"  prop="confirmPassword">
                <el-input v-model="dataInfo.confirmPassword" class="cms-width" type="password"></el-input>
            </el-form-item>
             <el-form-item label="会员组" class="flex-50" prop="groupId">
                <el-select v-model="dataInfo.groupId" class="cms-width">
                <el-option 
                v-for="(item,index) in memberGroup" 
                :key="index"
                :label="item.name"
                :value="item.id"
                ></el-option>
                </el-select>
            </el-form-item>  
            <el-form-item label="等级" class="flex-50"  prop="rank">
                <el-input v-model="dataInfo.rank" class="cms-width" type="number"></el-input>
                <span class="gray">越大等级越高</span> 
            </el-form-item>
            <el-form-item label="真实姓名" class="flex-50"  prop="realname">
                <el-input v-model="dataInfo.realname" class="cms-width" type="username"></el-input> 
            </el-form-item>
            <el-form-item label="性别" class="flex-50"  prop="gender">
                <el-radio-group v-model="dataInfo.gender">
                    <el-radio :label="true">男</el-radio>
                    <el-radio :label="false">女</el-radio>
                </el-radio-group> 
            </el-form-item>
            <el-form-item label="受限管理员" class="flex-100"  prop="selfAdmin">
                <el-radio-group v-model="dataInfo.selfAdmin">
                    <el-radio :label="true">是</el-radio>
                    <el-radio :label="false">否</el-radio>
                </el-radio-group> 
                <span class="gray">只能管理自己的数据</span> 
            </el-form-item>
            <el-form-item label="角色" class="flex-100"  prop="roleIds">
                <el-checkbox-group v-model="dataInfo.roleIds" @change="getRoleIds">
                    <el-checkbox v-for="(item,index) in roles" :key="index" :label="item.id">{{item.name}}</el-checkbox>
                </el-checkbox-group>
                <span class="gray">功能权限的控制在角色中</span> 
            </el-form-item>

            <el-form-item label="部门" class="flex-50" >      
                  <!-- 部门选择 -->
                 <el-cascader
                :options="departmentList"
                 v-model="parents"
                  :props="defaultProps"
                  :change-on-select="true"
                 class="cms-width"></el-cascader>
                <span class="gray">内容管理和栏目管理权限集中控制在部门管理中</span> 
            </el-form-item>


            <el-form-item label="数据权限" class="flex-100">
                    <el-checkbox v-model="dataInfo.allChannels">所有栏目内容权限</el-checkbox>
                    <el-checkbox v-model="dataInfo.allControlChannels">所有栏目控制权限</el-checkbox>
                <span class="gray">功能权限的控制在角色中</span> 
            </el-form-item>


            <div class="form-footer">
                <el-button type="warning" @click="add(true)"
                    v-perms="'/adminGlocal/add'"
                >保存并继续添加</el-button>
                <el-button type="primary"  @click="add(false)"
                    v-perms="'/adminGlocal/add'"
                >提交</el-button>
                <el-button type="info" @click="$reset">重置</el-button>
         </div>
    </el-form>
</section>
</template>
<script>
import listMixins from '@/mixins/form';
import axios from "axios";
import api from "@/api/api";
import va from "@/rules";
export default {
    mixins:[listMixins],
  data() {
       function password() {
      return {
        validator: (rule, value, callback) => {
          if (value === "") {
            callback();
          } else {
            if (self.dataInfo.confirmPassword !== "") {
              self.$refs.form.validateField("confirmPassword");
            }
            callback();
          }
        },
        trigger: "blur"
      };
    }
    function confirmPassword() {
      return {
        validator: (rule, value, callback) => {
          if (value === "") {
            callback();
          } else if (value !== self.dataInfo.password) {
            callback(new Error('前后密码不一致'));
          } else {
            callback();
          }
        },
        trigger: "blur"
      };
    }
    function rank() {//用户名验证
    return {
        validator: (rule, value, callback) => {
                if(value!=''){
                         axios.post(api.adminValRank,{id:'',rank:value}).then(res=>{
                if(value==''){
                     callback();    
                }else{
                    if(res.code=='200'){
                    if (!res.body.result){
                        callback(new Error('不能大于自身级别'))
                    }else{
                        callback();      
                    } 
                }else{
                    callback(new Error('服务器响应异常'));  
                }
                }
                      
            })
         }   
        }, trigger: 'blur'
      }
   }
    let self = this;
    let required = va.required('此项必填');
    let number = va.number('只能输入数字');
    let email = va.email('请输入正确的邮箱地址');
    let validateName = va.validateName('用户名已存在');
    let tel = va.tel('请输入正确的固定电话');
    let mobile = va.mobile('请输入正确的手机号'); 
    return {
      params: {//只需要业务参数
        queryUsername: "",
        https:""
      },
      rules: {
        //校验规则
        username: [required],
        email: [email],
        password: [password(),required],
        confirmPassword: [confirmPassword(),required],
        groupId: [required, number],
        grain: [required, number],
        phone: [tel],
        mobile: [mobile],
        disabled:[required],
        rank:[number,rank()],
      },
      memberGroup: [], //业务变量会员组
      roles:[],
      dialogDepartments: false, //部门触发按钮
      parents:[],//层级选择器
      departmentList: [],//部门列表
      defaultProps: {
        label: "name",
        value: "id",
        children: "child"
      },
      dialogVisible:false
    };
  },
     methods:{
         getRoleIds(value){
               console.log(value);
               
         },
      getDataInfo(id,https){
        let api = this.$api; //API地址
        axios
        .all([
             axios.post(api.adminGlocalGet, {id:id,https:https}), //axios批量发送请求
             axios.post(api.groupList),
             axios.post(api.roleList),
             axios.post(api.departmentList),
        ])
        .then(
            axios.spread((userGlocal,groups,roles,departmentList)=>{
                this.dataInfo=userGlocal.body;
                this.dataInfo.groupId=1;
                this.dataInfo.gender=true;
                this.dataInfo.rank=1;
                this.memberGroup=groups.body;
                this.roles=roles.body;
                this.dataInfo.allChannels=this.dataInfo.allChannels[0];
                this.dataInfo.allControlChannels=this.dataInfo.allControlChannels[0];
                this.departmentList =this.departmentList.concat({name: '无', pid: '', id: '', priority: '', weights: ''});
                this.departmentList = this.departmentList.concat(departmentList.body);              
                console.log(this.departmentList)
                this.$refs["form"].resetFields();
                this.loading = false;
            })
        )
        .catch(err => {
          this.loading = false;
        });
      },
        add(state) {
             let res={};              
            for(let i in this.dataInfo){
                res[i]=this.dataInfo[i]
            }
            res.departmentId=this.parents[this.parents.length-1];
            res.roleIds=res.roleIds.join(',');
              res.steps=1;
             for(let i in res){
                if((typeof res[i])=='object'){
                    delete res[i]
                }
             }      
             this.saveDataInfo(state,this.$api.adminGlocalSave,res, "list");
        }
        
  },
  created(){
    //初始获取数据
    this.getDataInfo(this.id);
  }
};
</script>

<style>

</style>


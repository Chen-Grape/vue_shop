<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>


        <!-- 卡片  -->
        <el-card>
            <!-- 搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容"  class="input-with-select"
                        v-model="queryInfo.query" clearable @clear="getUserList">
                       <el-button slot="append" 
                        icon="el-icon-search"
                         @click="getUserList"></el-button>
                    </el-input>    
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 用户列表区域 -->

            <el-table :data="userlist" border  stripe>
                <el-table-column label="#" type="index" ></el-table-column> 
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态" >
                    <!-- 作用域插槽渲染按钮 -->
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" >
                    <template slot-scope="scope" >
                        <!-- 修改按钮 -->
                        <el-button type="primary" icon="el-icon-edit" 
                        size="mini" @click="showEditDialog(scope.row.id)" class="btn"></el-button>
                       <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini"
                        @click="removeUserById(scope.row.id)" class="btn"></el-button>
                       <!-- 分配角色 -->
                        <el-tooltip  effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini" class="btn" @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <!-- 分页区域 -->
            <el-pagination
                  @size-change="handleSizeChange"
                  @current-change="handleCurrentChange"
                  :current-page="queryInfo.pagenum"
                  :page-sizes="[1, 2, 5, 10]"
                  :page-size="queryInfo.pagesize"
                  layout="total, sizes, prev, pager, next, jumper"
                  :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加用户的对话框 -->
       <el-dialog
          title="添加用户"
          :visible.sync="addDialogVisible"
          width="50%"
          @close="addDialogClosed">
            <!-- 内容主体区域 -->
            <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="70px">
                <el-form-item label="用户名" prop="username">
                  <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
               </el-form-item>
            </el-form>
            <!-- 内容的底部区 -->
            <span slot="footer" class="dialog-footer">
              <el-button @click="addDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
       </el-dialog>
       <!-- 修改用户对话框 -->
       <el-dialog
          title="编辑用户"
          :visible.sync="editDialogVisible"
           width="50%" @close="editDialogCloserd"
           >
              <el-form :model="editForm" :rules="editFormRules"
               ref="editFormRef" label-width="70px" >
                  <el-form-item label="用户名" >
                      <el-input v-model="editForm.username" disabled></el-input>
                 </el-form-item>
                 <!-- 提供校验规则需要添加prop -->
                 <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                 </el-form-item>
                 <el-form-item label="电话" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
               </el-form-item>
             </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
         </span>
      </el-dialog>

      <!-- 分配角色对话框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="setRoleDialogVisible"
        width="50%" @close="setRoleDialogCloed"
        >
       <div>
           <p>当前的用户: {{userInfo.username}}</p>
           <p>当前的角色: {{userInfo.role_name}}</p>
           <p>分配新角色: 
             <el-select v-model="selectRoleId" placeholder="请选择">
                <el-option
                  v-for="item in rolesList"
                  :key="item.id"
                  :label="item.roleName"
                  :value="item.id">
                </el-option>
              </el-select>
           </p>
       </div>
       <span slot="footer" class="dialog-footer">
         <el-button @click="setRoleDialogVisible = false">取 消</el-button>
         <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
     </el-dialog>
    </div>
</template>


<script>
export default{
    data() {
        //验证邮箱的剪头函数 规则 规则，参数，回调函数
        var checkEmail = (rule ,value,cb)=>{
            //验证邮箱的正则表达式
            const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
            if(regEmail.test(value)){
                //合法邮箱
                return cb()
            }
            cb(new Error('请输入合法的邮箱'))
        }
        //验证手机号规则
        var checkMobile = (rule ,value,cb)=>{
            //验证手机号的正则表达式
            const regMoblie =  /^1[3-9]\d{9}$/
            
            if(regMoblie.test(value)){
                //合法邮箱
                return cb()
            }
            cb(new Error('请输入合法的手机号'))
    
        }
        return {
            //获取对象的参数对象
            queryInfo:{
                query:'',
                //当前的页数
                pagenum:1,
                //当前每页显示多少条数据
                pagesize:2
            },
            userlist:[],
            total:0,
            //控制对话框的显示与隐藏
            addDialogVisible:false,
            editDialogVisible:false,
            setRoleDialogVisible:false,
            //添加用户表单的数据
            addForm:{
                username:'',
                password:'',
                email:'',
                mobile:''
            },
            editForm:{},
            //添加表单验证规则
            addFormRules:{
                username:[
                    {required:true,message:'请输入用户名称',trigger:'blur'},
                    {min:3,max:10,message:'用户名长度在3-10字符之间',trigger:'blur'}
      
                ],
                password:[
                    {required:true,message:'请输入密码',trigger:'blur'},
                    {min:6,max:15,message:'密码长度在6-15字符之间',trigger:'blur'}
      
                ],
                email:[
                {required:true,message:'请输入邮箱',trigger:'blur'},
                {validator:checkEmail,trigger:'blur'}
                ],
                mobile:[
                {required:true,message:'请输入手机',trigger:'blur'},
                {validator:checkMobile,trigger:'blur'}
                ]
            },
            
            editFormRules:{
                email:[
                {required:true,message:'请输入邮箱',trigger:'blur'},
                {validator:checkEmail,trigger:'blur'}
                ],
                mobile:[
                {required:true,message:'请输入手机',trigger:'blur'},
                {validator:checkMobile,trigger:'blur'}
                ]
            },
            //需要被分配角色的用户信息
            userInfo:{},
            rolesList:[],
            //已选中的角色Id
            selectRoleId:''
        }
    },
    created() {
        this.getUserList()
    },
    methods: {
        async getUserList(){
            // 解构重命名为res
         const{ data: res } = await  this.$http.get('users',{
             params: this.queryInfo})
         if(res.meta.status !==200) {return this.$message.error('获取用户列表失败！')}
         this.userlist = res.data.users
         this.total =res.data.total
        //  console.log("-----res"+res)
        //  console.log("-----res.data"+res.data)
        },
        ///监听pagesize事件
        handleSizeChange(newSize){
            // console.log(newSize)
            this.queryInfo.pagesize = newSize
            ///获取分页参数重新请求
            this.getUserList()
        },
        //监听页码值 改变的事件
        handleCurrentChange(newPage){
            // console.log(newPage)
            this.queryInfo.pagenum = newPage
            this.getUserList()
        },
        //监听swtich开关的状态
        async  userStateChanged(userInfo){
          console.log(userInfo)
          const {data: res} = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
          if(res.meta.status !== 200){
            userInfo.mg_state = !userInfo.mg_state
              return this.$message.error('更新用户状态失败')
          }  
          this.$message.success('更新用户状态成功')
        },
        //监听添加用户对话框的关闭事件
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        //点击按钮，添加新用户
        addUser(){
            this.$refs.addFormRef.validate(async valid =>{
                //如果校验失败
                if(!valid) return
                //可以发起添加用户请求

               const{data: res} = await this.$http.post('users',this.addForm)
                
                if(res.meta.status !== 201){
                    this.$message.error('添加用户失败！')
                }
                this.$message.success('添加用户成功！')
                //隐藏添加用户的对话框
                this.addDialogVisible = false
                //重新获取用户的列表数据
                this.getUserList()
            })
        },
        //展示编辑用的对话框
        async showEditDialog(id){
            // console.log(id)
           const{data: res} = await this.$http.get('users/'+id)
            if(res.meta.status !==200){
                return this.$message.error('查询用户信息失败')
            }
            this.editForm = res.data
            this.editDialogVisible = true
        },
          //监听修改用户对话框的关闭事件
        editDialogCloserd(){
                this.$refs.editFormRef.resetFields()
            },
        editUserInfo(){
                this.$refs.editFormRef.validate(async valid=>{
                    if(!valid) return
                    //发起修改用户信息的数据请求
                    const{data :res} = await this.$http.put(
                        'users/'+this.editForm.id,
                        {
                            email:this.editForm.email,
                            mobile:this.editForm.mobile
                        }
                        )

                    if (res.meta.status !==200 ) {
                        return this.$message.error('更新用户信息失败')
                    }
                    //关闭对话框
                    this.editDialogVisible = false
                    //刷新数据了表
                    this.getUserList()
                    //提示修改成功
                    this.$message.success('更新用户信息成功')
                })
            
            },
            //根据id删除对应的用户信息
        async removeUserById(id){
                // console.log(id)
               const confirmResult= await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                //type指定前面的小图标
                type: 'warning'
             }).catch(err => err)
             //如果用户确认删除，返回值为字符串 confirm
             //如果用户取消了删除，则返回值为字符串 cancel

             //  console.log(confirmResult)
             if(confirmResult !== 'confirm'){
                return this.$message.info('已取消删除')
              }
             // console.log('确认了删除')
             const{data :res } =await this.$http.delete('users/'+id
             )

             if (res.meta.status !== 200) {
                return this.$message.error('删除用户失败')
                 }

              this.$message.success('删除用户成功')
              this.getUserList()
            },
        async setRole(userInfo){
            this.userInfo = userInfo

            //在展示对话框之前，获取所有角色的列表
            const{data:res} = await this.$http.get('roles')
            if(res.meta.status !==200){
                return this.$message.error("获取角色列表失败！")
            }
            this.rolesList = res.data

            this.setRoleDialogVisible = true
        },
        //点击按钮分配角色
       async saveRoleInfo(){
            if(!this.selectRoleId){
                return this.$message.error('请选择要分配的角色')
            }
            const { data : res} = await this.$http.put(
                `users/${this.userInfo.id}/role`,
                {
                     rid:this.selectRoleId
                }
            )
            if(res.meta.status!== 200){
                return this.$message.error('设置角色失败!')
            }
            this.setRoleDialogVisible = false
           
            this.getUserList()
            return this.$message.success('设置角色成功！')
        },
        //监听分配加色对话框的关闭事件
        setRoleDialogCloed(){
            this.selectRoleId =''
            this.userInfo = {}
        }
    }
}

</script>

<!--lang="less" scoped 防止组件样式冲突  -->
<style lang="less" scoped>
    .btn {
        width: 40px;
    }
</style>
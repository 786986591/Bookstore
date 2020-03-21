<template>
<div id="login">
      <div class="login-wrap">
        <ul class="menu-tab">
    <li v-for="item in menuTab" :key="item.id" :class="{'current': item.current}" @click="toggleMenu(item)">{{item.txt}}</li>
        </ul>
        <!--表单start-->
      <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" class="login-form" size="medium">
        <el-form-item prop="email" class="item-from">
          <label for="email">邮箱</label>
          <el-input id="email" type="text" v-model="ruleForm.email" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item prop="password" class="item-from">
          <label for="password" class="login-btn">密码</label>
          <el-input id="password" type="password" v-model="ruleForm.password" autocomplete="off" minlength="6" maxlength="20"></el-input>
        </el-form-item>

        <el-form-item prop="passwords" class="item-from" v-if=" model === 'register'">
          <label class="login-btn">重复密码</label>
          <el-input type="password" v-model="ruleForm.passwords" autocomplete="off" minlength="6" maxlength="20"></el-input>
        </el-form-item>

           <el-form-item prop="username" class="item-from" v-if=" model === 'register'">
          <label class="login-btn">昵称</label>
          <el-input type="text" v-model="ruleForm.username" autocomplete="off" minlength="6" maxlength="20"></el-input>
        </el-form-item>

           <el-form-item prop="mobile" class="item-from" v-if=" model === 'register'">
          <label class="login-btn">手机</label>
          <el-input type="text" v-model="ruleForm.mobile" autocomplete="off" minlength="6" maxlength="20"></el-input>
        </el-form-item>

      <el-form-item>
          <el-button type="danger" @click="submitForm('ruleForm')" class="login-btn block">提交</el-button>
      </el-form-item>
           </el-form>
</div>
</div>
</template>
<script>
import axios from 'axios'
import { Register, Login } from "@/api/login";
import {ref, reactive,isRef,toRefs,onMounted} from '@vue/composition-api'
import {stripscript,validateEmails,validatePass,validateMobilenum,validateUser} from '@/utils/validate';
export default{
  name:'login',
  setup(props,{refs}){
            var validateEmail = (rule, value, callback) => { 
        if (value === '') {
          callback(new Error('请输入邮箱'));
        } else if(validateEmails(value)){
          callback(new Error("邮箱格式错误"));
        } 
        else {
          callback();
        }  
      };
  
    let validatePassword = (rule, value, callback) => {
    ruleForm.password =stripscript(value)
    value= ruleForm.password
        if (value === '') {
          callback(new Error('请输入密码'));
        } else if (validatePass(value)) {
          callback(new Error('密码格式为6至20为数字加字母!'));
        } else { 
          callback();
        }
    }
    let validatePasswords = (rule, value, callback) => {
       ruleForm.passwords =stripscript(value)
       value= ruleForm.passwords
        if (value === '') {
          callback(new Error('请再次输入密码'));
        } else if (value != ruleForm.password) {
          callback(new Error('重复输入密码不正确!'));
        } else {
          callback();
        }
      };

    let validateUsername = (rule, value, callback) => { 
        if (value === '') {
          callback(new Error('请输入用户昵称'));
        }
        else {
          callback();
        }  };

    let validateMobile = (rule, value, callback) => { 
        if (value === '') {
          callback(new Error('请输入电话号码'));
        } else if(validateMobilenum(value)){
          callback(new Error("电话号码格式错误"));
        } 
        else {
          callback();
           } 
         };

    const  menuTab=reactive([
        { txt: '登录',current: true ,type :'login'},
        { txt: '注册',current: false ,type:'register'}
        ])
    
      //模块值
    const model = ref('login')

    const ruleForm = reactive ({
          email: '',
          password: '',
          mobile:'',
          username:'',
        })

    const rules= reactive({
          email: [
            { validator: validateEmail, trigger: 'blur' }
          ],
          password: [
            { validator: validatePassword, trigger: 'blur' }
          ],
            passwords: [
            { validator: validatePasswords, trigger: 'blur' }
          ],
            username: [
            { validator: validateUsername, trigger: 'blur' }
          ],
            mobile: [
            { validator: validateMobile, trigger: 'blur' }
          ],
        })
/**声明函数 */
    const  toggleMenu = (data => {
            menuTab.forEach((elem) => {
                elem.current =false                
            });
            //高光
            data.current= true
            //修改模块值
            model.value= data.type
        })

     const submitForm = (formName => {
        refs[formName].validate((valid) => {
          // 表单验证通过
          if (valid) {
            // 三元运算
            model.value === 'login' ? login() : register()
          } else {
            console.log('error submit!!');
            return false;
          }
        })
      }) 
//登录
      const login = (() => {
        let repuestData = {
          username: ruleForm.username,
          password: ruleForm.password,
        }
        root.$store.dispatch('app/login', repuestData).then(response => {
          // 页面跳转
          root.$router.push({
            name: 'Console'
          })
        }).catch(error => {});
      })


        const register = (() => {
        let requestData = {
          username: ruleForm.username,
          password: ruleForm.password,
          mobile:ruleForm.mobile,
          email:ruleForm.email,
          module: 'register'
        }
        // 注册接口
        Register(requestData).then(response => {
          let data = response.data
          root.$message({
            message: data.message,
            type: 'success'
          })
          // 模拟注册成功
          toggleMenu(menuTab[0])
          clearCountDown()
        }).catch(error => {
          // 失败时执行的代码
        })
      })
/**生命周期 */
//挂在完成后  
    onMounted (() =>{
        })

        return{
          menuTab,
          model,
          ruleForm,
          rules,
           toggleMenu,
           submitForm

        }
  },
    created(){},
}
</script>
<style lang="scss" scoped>
#login {
  height: 100vh;
  background-color: #344a5f;
}
.login-wrap {
  width: 330px;
  margin: auto;
}
.menu-tab {
  text-align: center;
  li {
    display: inline-block;
    width: 88px;
    line-height: 36px;
    font-size: 14px;
    color: #fff;
    border-radius: 2px;
    cursor: pointer;
  }
  .current {
    background-color: rgba(0, 0, 0, .1);
  }
}
.login-form{
 margin-top: 29px;
  label {
    display: block;
    margin-bottom: 3px;
    font-size: 14px;
    color: #fff;
    }
}
.item-form { margin-bottom: 13px; }
.block {
    display: block;
    width: 100%;
  }
.login-btn { margin-top: 20px; }
</style>
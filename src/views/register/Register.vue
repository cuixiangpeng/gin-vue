<template>
  <div class="register">
    <b-row class="mt-5">
      <b-col
        md="8"
        offset-md="2"
        lg="6"
        offset-lg="3"
      >
        <b-card title="注册">
          <b-form>
            <b-form-group label="姓名">
              <b-form-input
                v-model="$v.user.name.$model"
                type="text"
                placeholder="输入您名称(选填)"
              ></b-form-input>
            </b-form-group>
            <b-form-group label="手机号">
              <b-form-input
                v-model="$v.user.telephone.$model"
                type="number"
                placeholder="输入您的手机号"
                :state="validationState('telephone')"
              ></b-form-input>
              <b-form-invalid-feedback :state="validationState('telephone')">
                手机号不符合要求
              </b-form-invalid-feedback>
            </b-form-group>
            <b-form-group label="密码">
              <b-form-input
                v-model="$v.user.password.$model"
                type="text"
                placeholder="输入您的密码"
                :state="validationState('password')"
              ></b-form-input>
              <b-form-invalid-feedback :state="validationState('password')">
                密码必须大于等于6位
              </b-form-invalid-feedback>
            </b-form-group>
            <b-form-group>
            <b-button
                @click="register"
                variant="outline-primary"
                block
              >注册</b-button>
            </b-form-group>
          </b-form>
        </b-card>
      </b-col>
    </b-row>
  </div>
</template>
<script>
import { required, minLength } from 'vuelidate/lib/validators';

import customValidator from '@/helper/validator';

import userService from '@/service/userService';

import { mapMutations } from 'vuex';

export default {
  data() {
    return {
      user: {
        name: '',
        telephone: '',
        password: '',
      },
    };
  },
  validations: {
    user: {
      name: {

      },
      telephone: {
        required,
        telephone: customValidator.telephoneValidator,

      },
      password: {
        required,
        minLength: minLength(6),

      },
    },
  },
  methods: {
    ...mapMutations('userModule', ['SET_TOKEN', 'SET_USERINFO']),
    validationState(name) {
      // 这里是es6 解构赋值
      const { $dirty, $error } = this.$v.user[name];
      return $dirty ? !$error : null;
    },
    register() {
      // 验证数据
      this.$v.user.$touch();
      if (this.$v.user.$anyError) {
        return;
      }
      // 请求
      userService.register(this.user).then((res) => {
        // 保存token
        this.SET_TOKEN(res.data.data.token);
        return userService.info();
      }).then((response) => {
        // 保存用户信息
        this.SET_USERINFO(response.data.data.user);
        // 跳转主页
        this.$router.replace({ name: 'Home' });
      }).catch((err) => {
        this.$bvToast.toast(err.response.data.msg, {
          title: '数据验证错误',
          variant: 'danger',
          solid: true,
        });
      });
      console.log('register');
    },
  },
};
</script>
<style lang="scss" scoped>
</style>

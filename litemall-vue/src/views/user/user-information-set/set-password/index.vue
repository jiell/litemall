<template>
  <div>
    <van-cell-group>

      <van-field
              label="新密码"
              v-model="password"
              type="password"
              placeholder="请输入新密码"
      />

      <!--			<van-field-->
      <!--				label="验证码"-->
      <!--				v-model="code"-->
      <!--				@click-icon="getCode"-->
      <!--				placeholder="请输入验证码">-->

      <span slot="icon"
            class="verifi_code red"
            :class="{verifi_code_counting: counting}"
            @click="getCode">
					<countdown v-if="counting" :time="60000" @end="countdownend">
					  <template slot-scope="props">{{ +props.seconds || 60 }}秒后获取</template>
					</countdown>
					<span v-else>获取验证码</span>
				</span>
      </van-field>
    </van-cell-group>

    <div class="bottom_btn">
      <van-button size="large" type="danger" @click="modifypassword">保存</van-button>
    </div>
  </div>
</template>


<script>
  import {authCaptcha, authReset, authLogout} from '@/api/api';
  import {removeLocalStorage} from '@/utils/local-storage';
  import {Field} from 'vant';

  export default {
    data: () => ({
      password: '',
      mobile: '',
      code: '',
      counting: false
    }),

    methods: {
      modifypassword() {
        if (this.passwordValid()) {
          authReset({
            password: this.password,
            // mobile: this.mobile
          })
              .then(() => {
                this.$dialog.alert({message: '保存成功, 请重新登录.'})
                authLogout();
                this.$router.push({name: 'login'})
              });
        }
      },
      passwordValid() {
        if (!this.password) {
          this.$toast('密码不能为空')
          return false
        }
        return true;
      },
      getCode() {
        if (this.mobile === '') {
          this.$toast.fail('请输入号码');
          return
        }

        if (!this.counting) {
          authCaptcha({
            mobile: this.mobile,
            type: 'change-password'
          }).then(() => {
            this.$toast.success('发送成功');
            this.counting = true;
          }).catch(error => {
            this.$toast.fail(error.data.errmsg);
            this.counting = false;
          })

        }
      },
    },

    components: {
      [Field.name]: Field
    }
  };
</script>


<style lang="scss" scoped>
  @import '../../../../assets/scss/var';
  @import '../../../../assets/scss/mixin';

  .bottom_btn {
    padding: 30px 15px 0 15px;
  }

  .verifi_code {
    @include one-border;
    padding-left: 10px;

    &::after {
      border-bottom: 0;
      border-left: 1px solid $border-color;
    }

    &_counting {
      color: $font-color-gray;
    }
  }
</style>

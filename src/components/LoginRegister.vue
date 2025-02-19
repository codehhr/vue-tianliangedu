<template>
  <!-- 登录注册入口 start -->
  <div class="login-register">
    <div class="login-register-entrance">
      <!-- 购物车 -->
      <a-icon class="shopping-cart" type="shopping-cart" />
      <!-- 分割线 -->
      <a-divider type="vertical" class="divider" />
      <!-- 登录/注册 -->
      <a-dropdown v-show="alreadyLogin" class="user-drop-menu">
        <a class="ant-dropdown-link" @click="(e) => e.preventDefault()">
          {{ $store.state.userInfo ? $store.state.userInfo.nickname : "" }}
        </a>
        <a-menu slot="overlay">
          <a-menu-item>
            <router-link :to="{ name: 'PersonalCenter' }">个人中心</router-link>
          </a-menu-item>
          <a-menu-item>
            <a @click="logOut">退出登录</a>
          </a-menu-item>
        </a-menu>
      </a-dropdown>

      <!-- 登录/注册 按钮 -->
      <a-button
        v-show="!alreadyLogin"
        class="login-register-btn"
        type="primary"
        @click="showLoginModal"
      >
        登录 / 注册
      </a-button>

      <!-- 头像 -->
      <a-avatar class="avatar" :src="userInfo ? userInfo.avatarUrl : ''" />

      <!-- 登录 modal start -->
      <a-modal
        v-model="loginVisible"
        :closable="false"
        :footer="null"
        :keyboard="false"
        :maskClosable="false"
        style="text-align:center;"
      >
        <a-icon @click="closeModal" class="modal-close-btn" type="close" />

        <a-tabs default-active-key="1">
          <a-tab-pane key="1" tab="普通登录">
            <a-form
              id="components-form-normal-login"
              :form="normalLoginForm"
              class="login-form"
              @submit="handleLoginSubmit"
            >
              <a-form-item>
                <a-input
                  v-decorator="[
                    'username',
                    {
                      rules: [{ required: true, message: '请输入用户名' }],
                    },
                  ]"
                  placeholder="用户名"
                  autocomplete
                >
                  <a-icon
                    slot="prefix"
                    type="user"
                    style="color: rgba(0,0,0,.25)"
                  />
                </a-input>
              </a-form-item>
              <a-form-item>
                <a-input
                  v-decorator="[
                    'password',
                    {
                      rules: [{ required: true, message: '请输入密码' }],
                    },
                  ]"
                  type="password"
                  placeholder="密码"
                  autocomplete
                >
                  <a-icon
                    slot="prefix"
                    type="lock"
                    style="color: rgba(0,0,0,.25)"
                  />
                </a-input>
              </a-form-item>
              <a-form-item>
                <a-button
                  type="primary"
                  html-type="submit"
                  class="login-form-button"
                >
                  登录
                </a-button>

                <div class="login-form-footer">
                  <a-button
                    class="go-to-register"
                    type="primary"
                    @click="goToRegister"
                  >
                    立即注册
                  </a-button>
                  <a class="login-form-forgot" href="">
                    忘记密码?
                  </a>
                </div>
              </a-form-item>
            </a-form>
          </a-tab-pane>
          <a-tab-pane key="2" tab="手机登录">
            <a-form
              id="components-form-tel-login"
              :form="telLoginForm"
              class="login-form"
              @submit="handleSubmitWithTel"
            >
              <a-form-item>
                <a-input
                  v-decorator="[
                    'mobile',
                    {
                      rules: [{ required: true, message: '请输入手机号' }],
                    },
                  ]"
                  placeholder="手机号"
                  autocomplete
                >
                  <a-icon
                    slot="prefix"
                    type="user"
                    style="color: rgba(0,0,0,.25)"
                  />
                </a-input>
              </a-form-item>
              <a-form-item v-bind="formItemLayout">
                <a-row :gutter="24">
                  <a-col :span="20">
                    <a-input
                      v-decorator="[
                        'code',
                        {
                          rules: [
                            {
                              message: '请输入验证码!',
                            },
                          ],
                        },
                      ]"
                      autocomplete
                      placeholder="验证码"
                      title="请输入验证码"
                    />
                  </a-col>
                  <a-col :span="4">
                    <a-button
                      :disabled="loginCodeBtnDisabled"
                      class="get-code"
                      @click="getLoginMobileCode"
                    >
                      <span v-show="!hadRequestLoginCode">获取验证码</span>
                      <span v-show="hadRequestLoginCode">{{
                        loginSecond
                      }}</span>
                    </a-button>
                  </a-col>
                </a-row>
              </a-form-item>
              <a-form-item>
                <a-button
                  type="primary"
                  html-type="submit"
                  class="login-form-button"
                >
                  登录
                </a-button>

                <div class="login-form-footer">
                  <a-button
                    class="go-to-register"
                    type="primary"
                    @click="goToRegister"
                  >
                    立即注册
                  </a-button>
                </div>
              </a-form-item>
            </a-form>
          </a-tab-pane>
        </a-tabs>
      </a-modal>
      <!-- 登录 modal end -->

      <!-- 注册 modal start -->
      <a-modal
        v-model="registerVisible"
        :closable="false"
        :footer="null"
        :keyboard="false"
        :maskClosable="false"
        title="注 册"
        style="text-align:center;"
      >
        <a-icon @click="closeModal" class="modal-close-btn" type="close" />
        <a-form :form="registerForm" @submit="handleRegisterSubmit">
          <a-form-item v-bind="formItemLayout" label="帐号">
            <a-input
              v-decorator="[
                'loginName',
                {
                  rules: [{ required: true, message: '请输入用户名' }],
                },
              ]"
              title="请输入账号(6-20位，非纯数字)!"
              placeholder="请输入账号(6-20位，非纯数字)!"
              autocomplete
            >
            </a-input>
          </a-form-item>
          <a-form-item v-bind="formItemLayout" label="密码" has-feedback>
            <a-input
              v-decorator="[
                'password',
                {
                  rules: [
                    {
                      required: true,
                      message: '请输入你的密码(6-20位，非汉字)!',
                    },
                    {
                      validator: validateToNextPassword,
                    },
                  ],
                },
              ]"
              title="请输入你的密码(6-20位，非汉字)!"
              placeholder="请输入你的密码(6-20位，非汉字)!"
              typeautocomplete="password"
            />
          </a-form-item>
          <a-form-item v-bind="formItemLayout" label="确认密码" has-feedback>
            <a-input
              v-decorator="[
                'confirm',
                {
                  rules: [
                    {
                      required: true,
                      message: '请再次输入你的密码!',
                    },
                    {
                      validator: compareToFirstPassword,
                    },
                  ],
                },
              ]"
              type="password"
              title="请再次输入你的密码"
              placeholder="请再次输入你的密码"
              autocomplete
              @blur="handleConfirmBlur"
            />
          </a-form-item>
          <a-form-item v-bind="formItemLayout">
            <span slot="label">
              昵称&nbsp;
              <a-tooltip title="其他人会看到这个名字">
                <a-icon type="question-circle-o" />
              </a-tooltip>
            </span>
            <a-input
              v-decorator="[
                'nickname',
                {
                  rules: [
                    {
                      required: true,
                      message: '请输入你的昵称!',
                      whitespace: true,
                    },
                  ],
                },
              ]"
              title="请输入你的昵称"
              placeholder="请输入你的昵称"
              autocomplete
            />
          </a-form-item>
          <a-form-item v-bind="formItemLayout" label="手机号码">
            <a-input
              v-decorator="[
                'mobile',
                {
                  rules: [
                    {
                      required: true,
                      message: '请输入你的手机号!',
                    },
                  ],
                },
              ]"
              autocomplete
              placeholder="请输入你的手机号"
              title="请输入你的手机号"
            >
              <a-select
                slot="addonBefore"
                v-decorator="['prefix', { initialValue: '86' }]"
                style="width: 70px;"
              >
                <a-select-option value="86">
                  +86
                </a-select-option>
                <a-select-option value="87">
                  +87
                </a-select-option>
              </a-select>
            </a-input>
          </a-form-item>
          <a-form-item v-bind="formItemLayout" label="验证码">
            <a-row :gutter="8">
              <a-col :span="12">
                <a-input
                  v-decorator="[
                    'code',
                    {
                      rules: [
                        {
                          message: '请输入验证码!',
                        },
                      ],
                    },
                  ]"
                  autocomplete
                  placeholder="请输入验证码"
                  title="请输入验证码"
                />
              </a-col>
              <a-col :span="12">
                <a-button
                  :disabled="registerCodeBtnDisabled"
                  class="get-code"
                  @click="getRegisterMobileCode"
                >
                  <span v-show="!hadRequestRegisterCode">获取验证码</span>
                  <span v-show="hadRequestRegisterCode">{{
                    registerSecond
                  }}</span>
                </a-button>
              </a-col>
            </a-row>
          </a-form-item>
          <a-button class="register" type="primary" html-type="submit">
            注册
          </a-button>
          <a-button class="go-to-login" type="primary" @click="goToLogin">
            去登录
          </a-button>
        </a-form>
      </a-modal>
      <!-- 注册 modal end -->
    </div>
  </div>
  <!-- 登录注册入口 end -->
</template>

<script>
import { mapState } from "vuex";
import {
  login,
  loginWithTel,
  getLoginCode,
  getRegisterCode,
  register,
  logOut,
} from "../api/api";

export default {
  name: "LoginRegister",
  data() {
    return {
      // 注册 start
      confirmDirty: false,
      autoCompleteResult: [],
      formItemLayout: {
        labelCol: {
          xs: { span: 24 },
          sm: { span: 8 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
      },
      tailFormItemLayout: {
        wrapperCol: {
          xs: {
            span: 24,
            offset: 0,
          },
          sm: {
            span: 16,
            offset: 8,
          },
        },
      },
      // 注册 end

      // 登录验证码定时器
      loginCodeInterval: null,
      // 获取验证码按钮禁用
      loginCodeBtnDisabled: false,
      // 已经请求验证码
      hadRequestLoginCode: false,
      // 请求验证码倒计时
      loginSecond: 1 * 60,

      // 获取注册验证码定时器
      registerCodeInterval: null,
      // 获取验证码按钮禁用
      registerCodeBtnDisabled: false,
      // 已经请求验证码
      hadRequestRegisterCode: false,
      // 请求验证码倒计时
      registerSecond: 1 * 60,
    };
  },
  methods: {
    // 显示登录模态框
    showLoginModal() {
      this.$store.commit("setLoginVisible", true);
    },
    //
    closeModal() {
      this.$store.commit("setLoginVisible", false);
      this.$store.commit("setRegisterVisible", false);
    },
    // 去注册
    goToRegister() {
      this.$store.commit("setLoginVisible", false);
      this.$store.commit("setRegisterVisible", true);
    },
    // 去登录
    goToLogin() {
      this.$store.commit("setRegisterVisible", false);
      this.$store.commit("setLoginVisible", true);
    },
    // 普通登录
    handleLoginSubmit(e) {
      e.preventDefault();
      this.normalLoginForm.validateFields((err, values) => {
        if (!err) {
          login(values.username, values.password).then((res) => {
            if (res.code === 0) {
              this.$store.commit("setLoginVisible", false);
              this.$message.success("登录成功");
              this.$store.commit("setUserInfoAndLoginStatus", {
                alreadyLogin: true,
                userInfo: res.userInfo,
              });
            } else {
              this.$message.warning("帐号或密码不正确");
            }
          });
        } else {
          this.$message.warning("请填写完整 ~");
        }
      });
    },
    // 手机登录
    handleSubmitWithTel(e) {
      e.preventDefault();
      this.telLoginForm.validateFields((err, values) => {
        if (!err) {
          loginWithTel(values.mobile, values.code).then((res) => {
            if (res.code === 0) {
              this.$store.commit("setLoginVisible", false);
              this.$message.success("登录成功");
              this.$store.commit("setUserInfoAndLoginStatus", {
                alreadyLogin: true,
                userInfo: res.userInfo,
              });
            } else {
              this.$message.warning("填写不正确 ~");
            }
          });
        } else {
          this.$message.warning("请填写完整 ~");
        }
      });
    },
    // 获取登录验证码
    getLoginMobileCode() {
      let that = this;
      // 获取验证码倒计时
      function countDown() {
        // 按钮禁用
        that.loginCodeBtnDisabled = true;

        // 按钮倒计时 start
        that.hadRequestLoginCode = true;
        if (!that.loginCodeInterval) {
          that.loginCodeInterval = setInterval(() => {
            if (that.loginSecond <= 0) {
              clearInterval(that.loginCodeInterval);
              that.loginCodeBtnDisabled = false;
              that.hadRequestLoginCode = false;
            } else {
              that.loginSecond--;
            }
          }, 1000);
        }
        // 按钮倒计时 end
      }

      // 获取验证码
      this.telLoginForm.validateFieldsAndScroll((err, values) => {
        if (!err) {
          getLoginCode(values.mobile).then((res) => {
            if (res.code === 0) {
              countDown();
              this.$message.success("验证码已发送 ~")
            } else {
              if (res.code === 500) {
                this.$message.warning(res.msg);
              }
            }
          });
        } else {
          that.$message.warning("请输入手机号");
        }
      });
    },
    // 退出登录
    logOut() {
      logOut().then((res) => {
        if (res.code === 0) {
          this.$store.commit("setUserInfoAndLoginStatus", {
            alreadyLogin: false,
            userInfo: null,
          });
          if (this.$route.path === "/") {
            this.$message.success("已退出登录");
          } else {
            this.$message.success("已退出登录");
            this.$router.push({ path: "/" });
          }
        } else {
          this.$message.error("错误");
        }
      });
    },

    // 注册 start
    handleRegisterSubmit(e) {
      e.preventDefault();
      this.registerForm.validateFieldsAndScroll((err, values) => {
        if (!err) {
          register(
            values.loginName,
            values.password,
            values.nickname,
            values.mobile,
            values.code
          ).then((res) => {
            if (res.code === 0) {
              this.$message.success("注册成功");
            }
          });
        } else {
          this.$message.warning("请填写完整 ~");
        }
      });
    },

    handleConfirmBlur(e) {
      const value = e.target.value;
      this.confirmDirty = this.confirmDirty || !!value;
    },

    // 验证密码 start
    compareToFirstPassword(rule, value, callback) {
      const form = this.registerForm;
      if (value && value !== form.getFieldValue("password")) {
        callback("两次密码输入不一致!");
      } else {
        callback();
      }
    },

    validateToNextPassword(rule, value, callback) {
      const form = this.registerForm;
      if (value && this.confirmDirty) {
        form.validateFields(["confirm"], { force: true });
      }
      callback();
    },
    // 验证密码 end

    // 获取注册验证码
    getRegisterMobileCode() {
      let that = this;
      // 获取验证码倒计时
      function countDown() {
        // 按钮禁用
        that.registerCodeBtnDisabled = true;

        // 按钮倒计时 start
        that.hadRequestRegisterCode = true;
        if (!that.registerCodeInterval) {
          that.registerCodeInterval = setInterval(() => {
            if (that.registerSecond <= 0) {
              clearInterval(that.registerCodeInterval);
              that.registerCodeBtnDisabled = false;
              that.hadRequestRegisterCode = false;
            } else {
              that.registerSecond--;
            }
          }, 1000);
        }
        // 按钮倒计时 end
      }

      // 获取验证码
      this.registerForm.validateFieldsAndScroll((err, values) => {
        if (!err) {
          if (!values.mobile) {
            this.$message.warning("请填写完整 ~");
          } else {
            getRegisterCode(values.mobile).then((res) => {
              if (res.code === 0) {
                countDown();
              } else {
                if (res.msg.includes("nvalid")) {
                  this.$message.warning("请规范输入手机号");
                } else {
                  this.$message.warning(res.msg);
                }
              }
            });
          }
        }
      });
    },

    // 注册 end
  },

  computed: {
    ...mapState({
      alreadyLogin: "alreadyLogin",
      userInfo: "userInfo",
      loginVisible: "loginVisible",
      registerVisible: "registerVisible",
    }),
  },

  beforeCreate() {
    this.normalLoginForm = this.$form.createForm(this, {
      name: "normal_login",
    });
    this.telLoginForm = this.$form.createForm(this, { name: "normal_login" });
    this.registerForm = this.$form.createForm(this, { name: "register" });
  },
};
</script>

<style scoped lang="less">
.login-register {
  display: flex;
  align-items: center;
  justify-content: space-between;

  .login-register-entrance {
    .divider {
      margin-left: 10px;
    }

    .login-register-btn {
      padding: 0;
      width: 90px;
      background-color: transparent;
      color: #333333;
      border: none;
      box-shadow: none;
    }

    .user-drop-menu {
      width: 90px;
      color: #000000;
    }
  }
}

.login-form-button {
  width: 100%;
  height: 36px;
}

.login-form-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>

<style lang="less">
@main-color: #00cf8c;

.avatar {
  position: relative;
  top: -2px;
  margin: 0 10px;
  text-align: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;

  img {
    width: 100%;
    height: 100%;
  }
}

.ant-modal {
  padding: 0 !important;
  width: 400px !important;

  .ant-modal-content {
    position: relative;
    overflow: hidden;
    .modal-close-btn {
      position: absolute;
      top: 0;
      right: 0;
      width: 40px;
      height: 40px;
      line-height: 40px;
      cursor: pointer;
    }
    .ant-modal-body {
      padding: 10px 40px 0 !important;

      .go-to-register,
      .go-to-login {
        color: #000000a6;
        background-color: transparent;
        border: none;
        box-shadow: none;

        &:hover {
          color: @main-color;
        }
      }

      .ant-input-group .ant-input {
        border-radius: 0;
      }

      .get-code {
        min-width: 100px;
        color: #000000;
      }

      .register {
        width: 100%;
      }

      .go-to-login {
        margin: 10px;
        float: right;
      }
    }
  }
}
</style>

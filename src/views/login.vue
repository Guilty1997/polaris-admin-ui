<template>
  <div class="login">
    <div class="login-shell">
      <div class="hero-panel">
        <div class="hero-content">
          <span class="hero-chip">{{ title || 'Polaris Admin' }}</span>
          <h2 class="hero-title">更现代的后台体验</h2>
          <p class="hero-subtitle">双栏渐变设计、清晰的信息层次，让登录更高效。</p>
          <a class="hero-link" href="/" rel="noreferrer">{{ title || 'Polaris Admin' }}</a>
        </div>
      </div>
      <div class="form-panel">
        <div class="panel-header">
          <span />
          <lang-select />
        </div>
        <div class="title-box">
          <div>
            <h3 class="title">{{ proxy.$t('login.login') }}</h3>
            <p class="subtitle">请输入账号信息登录系统</p>
          </div>
        </div>

        <el-form ref="loginRef" :model="loginForm" :rules="loginRules" class="login-form">
          <el-form-item v-if="tenantEnabled" prop="tenantId">
            <el-select v-model="loginForm.tenantId" filterable :placeholder="proxy.$t('login.selectPlaceholder')" class="full-width">
              <el-option v-for="item in tenantList" :key="item.tenantId" :label="item.companyName" :value="item.tenantId"></el-option>
              <template #prefix><svg-icon icon-class="company" class="el-input__icon input-icon" /></template>
            </el-select>
          </el-form-item>
          <el-form-item prop="username">
            <el-input v-model="loginForm.username" type="text" size="large" auto-complete="off" :placeholder="proxy.$t('login.username')">
              <template #prefix><svg-icon icon-class="user" class="el-input__icon input-icon" /></template>
            </el-input>
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              v-model="loginForm.password"
              type="password"
              size="large"
              auto-complete="off"
              :placeholder="proxy.$t('login.password')"
              @keyup.enter="handleLogin"
            >
              <template #prefix><svg-icon icon-class="password" class="el-input__icon input-icon" /></template>
            </el-input>
          </el-form-item>
          <el-form-item v-if="captchaEnabled" prop="code" class="captcha-item">
            <el-input
              v-model="loginForm.code"
              size="large"
              auto-complete="off"
              :placeholder="proxy.$t('login.code')"
              class="code-input"
              @keyup.enter="handleLogin"
            >
              <template #prefix><svg-icon icon-class="validCode" class="el-input__icon input-icon" /></template>
            </el-input>
            <div class="login-code">
              <img :src="codeUrl" class="login-code-img" @click="getCode" />
            </div>
          </el-form-item>

          <div class="form-actions">
            <el-checkbox v-model="loginForm.rememberMe">{{ proxy.$t('login.rememberPassword') }}</el-checkbox>
            <div v-if="register" class="register-link">
              <router-link class="link-type" :to="'/register'">{{ proxy.$t('login.switchRegisterPage') }}</router-link>
            </div>
          </div>

          <el-form-item class="submit-item">
            <el-button :loading="loading" size="large" type="primary" class="full-width" @click.prevent="handleLogin">
              <span v-if="!loading">{{ proxy.$t('login.login') }}</span>
              <span v-else>{{ proxy.$t('login.logging') }}</span>
            </el-button>
          </el-form-item>
        </el-form>
        <div class="divider">
          <span>或使用社交账号登录</span>
        </div>
        <div class="social-login">
          <div class="social-buttons">
            <el-button circle :title="proxy.$t('login.social.wechat')" @click="doSocialLogin('wechat')">
              <svg-icon icon-class="wechat" />
            </el-button>
            <el-button circle :title="proxy.$t('login.social.maxkey')" @click="doSocialLogin('maxkey')">
              <svg-icon icon-class="maxkey" />
            </el-button>
            <el-button circle :title="proxy.$t('login.social.topiam')" @click="doSocialLogin('topiam')">
              <svg-icon icon-class="topiam" />
            </el-button>
            <el-button circle :title="proxy.$t('login.social.gitee')" @click="doSocialLogin('gitee')">
              <svg-icon icon-class="gitee" />
            </el-button>
            <el-button circle :title="proxy.$t('login.social.github')" @click="doSocialLogin('github')">
              <svg-icon icon-class="github" />
            </el-button>
          </div>
        </div>
        <div class="el-login-footer">
          <span>Copyright © 2018-2025 疯狂的狮子Li All Rights Reserved.</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { getCodeImg, getTenantList } from '@/api/login';
import { authBinding } from '@/api/system/social/auth';
import { useUserStore } from '@/store/modules/user';
import { LoginData, TenantVO } from '@/api/types';
import { to } from 'await-to-js';
import { HttpStatus } from '@/enums/RespEnum';
import { useI18n } from 'vue-i18n';

const { proxy } = getCurrentInstance() as ComponentInternalInstance;

const title = import.meta.env.VITE_APP_TITLE;
const userStore = useUserStore();
const router = useRouter();
const { t } = useI18n();

const loginForm = ref<LoginData>({
  tenantId: '000000',
  username: 'admin',
  password: 'admin123',
  rememberMe: false,
  code: '',
  uuid: ''
} as LoginData);

const loginRules: ElFormRules = {
  tenantId: [{ required: true, trigger: 'blur', message: t('login.rule.tenantId.required') }],
  username: [{ required: true, trigger: 'blur', message: t('login.rule.username.required') }],
  password: [{ required: true, trigger: 'blur', message: t('login.rule.password.required') }],
  code: [{ required: true, trigger: 'change', message: t('login.rule.code.required') }]
};

const codeUrl = ref('');
const loading = ref(false);
// 验证码开关
const captchaEnabled = ref(true);
// 租户开关
const tenantEnabled = ref(true);

// 注册开关
const register = ref(false);
const redirect = ref('/');
const loginRef = ref<ElFormInstance>();
// 租户列表
const tenantList = ref<TenantVO[]>([]);

watch(
  () => router.currentRoute.value,
  (newRoute: any) => {
    redirect.value = newRoute.query && newRoute.query.redirect && decodeURIComponent(newRoute.query.redirect);
  },
  { immediate: true }
);

const handleLogin = () => {
  loginRef.value?.validate(async (valid: boolean, fields: any) => {
    if (valid) {
      loading.value = true;
      // 勾选了需要记住密码设置在 localStorage 中设置记住用户名和密码
      if (loginForm.value.rememberMe) {
        localStorage.setItem('tenantId', String(loginForm.value.tenantId));
        localStorage.setItem('username', String(loginForm.value.username));
        localStorage.setItem('password', String(loginForm.value.password));
        localStorage.setItem('rememberMe', String(loginForm.value.rememberMe));
      } else {
        // 否则移除
        localStorage.removeItem('tenantId');
        localStorage.removeItem('username');
        localStorage.removeItem('password');
        localStorage.removeItem('rememberMe');
      }
      // 调用action的登录方法
      const [err] = await to(userStore.login(loginForm.value));
      if (!err) {
        const redirectUrl = redirect.value || '/';
        await router.push(redirectUrl);
        loading.value = false;
      } else {
        loading.value = false;
        // 重新获取验证码
        if (captchaEnabled.value) {
          await getCode();
        }
      }
    } else {
      console.log('error submit!', fields);
    }
  });
};

/**
 * 获取验证码
 */
const getCode = async () => {
  const res = await getCodeImg();
  const { data } = res;
  captchaEnabled.value = data.captchaEnabled === undefined ? true : data.captchaEnabled;
  if (captchaEnabled.value) {
    codeUrl.value = 'data:image/gif;base64,' + data.img;
    loginForm.value.uuid = data.uuid;
  }
};

const getLoginData = () => {
  const tenantId = localStorage.getItem('tenantId');
  const username = localStorage.getItem('username');
  const password = localStorage.getItem('password');
  const rememberMe = localStorage.getItem('rememberMe');
  loginForm.value = {
    tenantId: tenantId === null ? String(loginForm.value.tenantId) : tenantId,
    username: username === null ? String(loginForm.value.username) : username,
    password: password === null ? String(loginForm.value.password) : String(password),
    rememberMe: rememberMe === null ? false : Boolean(rememberMe)
  } as LoginData;
};

/**
 * 获取租户列表
 */
const initTenantList = async () => {
  const { data } = await getTenantList(false);
  tenantEnabled.value = data.tenantEnabled === undefined ? true : data.tenantEnabled;
  if (tenantEnabled.value) {
    tenantList.value = data.voList;
    if (tenantList.value != null && tenantList.value.length !== 0) {
      loginForm.value.tenantId = tenantList.value[0].tenantId;
    }
  }
};

/**
 * 第三方登录
 * @param type
 */
const doSocialLogin = (type: string) => {
  authBinding(type, loginForm.value.tenantId).then((res: any) => {
    if (res.code === HttpStatus.SUCCESS) {
      // 获取授权地址跳转
      window.location.href = res.data;
    } else {
      ElMessage.error(res.msg);
    }
  });
};

onMounted(() => {
  getCode();
  initTenantList();
  getLoginData();
});
</script>

<style lang="scss" scoped>
.login {
  min-height: 100vh;
  background: #f4f7fe;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.login-shell {
  width: 1100px;
  background: #ffffff;
  border-radius: 24px;
  box-shadow: 0 25px 50px rgba(59, 70, 98, 0.12);
  display: grid;
  grid-template-columns: 1fr 1fr;
  overflow: hidden;
}

.hero-panel {
  background: linear-gradient(135deg, #868cff 0%, #4318ff 100%);
  position: relative;
  display: flex;
  align-items: center;
  padding: 48px;
  color: #fff;
}

.hero-content {
  max-width: 360px;
}

.hero-chip {
  display: inline-flex;
  align-items: center;
  padding: 6px 12px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.18);
  font-weight: 600;
  font-size: 12px;
  letter-spacing: 0.5px;
}

.hero-title {
  margin: 18px 0 8px 0;
  font-size: 30px;
  font-weight: 700;
  line-height: 1.3;
}

.hero-subtitle {
  margin: 0 0 24px 0;
  color: rgba(255, 255, 255, 0.85);
  line-height: 1.6;
}

.hero-link {
  color: #ffffff;
  font-weight: 600;
  text-decoration: none;
}

.form-panel {
  padding: 40px 48px 32px;
  background: #ffffff;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  margin-bottom: 8px;

  :deep(.lang-select--style) {
    line-height: 0;
    color: #7483a3;
  }
}

.title-box {
  display: flex;
  align-items: center;
  justify-content: space-between;

  .title {
    margin: 0;
    color: #2b3674;
    font-size: 28px;
    font-weight: 700;
  }

  .subtitle {
    margin: 6px 0 0;
    color: #a3aed0;
    font-size: 14px;
  }
}

.social-login {
  display: flex;
  justify-content: center;
  padding: 6px 0 0;

  .social-buttons {
    display: flex;
    gap: 10px;

    :deep(.el-button) {
      border-color: #e0e5f2;
      color: #2b3674;
    }
  }
}

.divider {
  display: flex;
  align-items: center;
  gap: 12px;
  color: #a3aed0;
  font-size: 12px;

  &::before,
  &::after {
    content: '';
    flex: 1;
    height: 1px;
    background: #e0e5f2;
  }
}

.login-form {
  width: 100%;
  margin-top: 4px;

  .el-input {
    height: 44px;

    input {
      height: 44px;
    }
  }

  .input-icon {
    height: 20px;
    width: 16px;
    margin-left: 0;
  }
}

.full-width {
  width: 100%;
}

.captcha-item {
  display: flex;
  align-items: center;
  gap: 12px;

  .code-input {
    flex: 1;
  }
}

.login-code {
  height: 44px;
  display: flex;
  align-items: center;
  padding: 6px 10px;
  background: #f4f7fe;
  border-radius: 12px;
  border: 1px solid #e0e5f2;

  img {
    cursor: pointer;
    height: 100%;
  }
}

.form-actions {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 6px;
  margin-bottom: 4px;

  .register-link .link-type {
    color: #4318ff;
  }
}

.submit-item {
  margin-top: 4px;

  :deep(.el-button) {
    border-radius: 12px;
    height: 46px;
  }
}

.el-login-footer {
  margin-top: 8px;
  text-align: center;
  color: #a3aed0;
  font-size: 12px;
}

@media (max-width: 1024px) {
  .login-shell {
    grid-template-columns: 1fr;
  }

  .hero-panel {
    min-height: 220px;
  }
}
</style>

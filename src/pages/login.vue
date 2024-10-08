<template>
  <div class="flex min-h-screen items-center justify-center bg-white shadow-lg">
    <div class="w-full max-w-md space-y-8 p-8 bg-white shadow-lg rounded-md">
      <div>
        <h2 class="mt-6 text-center text-3xl font-bold text-gray-900">Sign in to your account</h2>
      </div>
      <form class="mt-8 space-y-6" @submit.prevent="handleLogin">
        <div class="rounded-md shadow-sm">
          <div>
            <label for="email" class="sr-only">Email address</label>
            <input id="username" name="username" type="email" required
              class="relative block w-full appearance-none rounded-md border border-gray-300 px-3 py-2 placeholder-gray-500 focus:z-10 focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm"
              placeholder="Email address" v-model="loginform.username" />
          </div>
          <div class="mt-4 relative">
            <label for="password" class="sr-only">Password</label>
            <input :type="showPassword ? 'text' : 'password'" id="password" name="password" required
              class="relative block w-full appearance-none rounded-md border border-gray-300 px-3 py-2 placeholder-gray-500 focus:z-10 focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm"
              placeholder="Password" v-model="loginform.password" />
            <span @click="togglePasswordVisibility"
              class="absolute inset-y-0 right-0 pr-3 flex items-center cursor-pointer">
              <i :class="showPassword ? 'fas fa-eye-slash' : 'fas fa-eye'"></i>
            </span>
          </div>
        </div>

        <div class="flex items-center justify-between">
          <div class="flex items-center">
            <input id="is_remember" name="is_remember" type="checkbox"
              class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
              v-model="loginform.is_remember" />
            <label for="is_remember" class="ml-2 block text-sm text-gray-900">Remember me</label>
          </div>

          <div class="text-sm">
            <a href="#" class="font-medium text-indigo-600 hover:text-indigo-500">Forgot your password?</a>
          </div>
        </div>

        <div>
          <button type="submit"
            class="group relative flex w-full justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
            Sign in
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { useAuthStore } from '~/stores/auth';
import authAPI from '@/utils/api/autheAPI';
import { toast } from 'vue3-toastify';

const authStore = useAuthStore();

const loginform = ref({
  username: '',
  password: '',
});
const UserType = {
  SUPERADMIN: 'SUPERADMIN',
  ADMIN: 'ADMIN',
  VENDOR: 'VENDOR',
  USER: 'USER',
};

const showPassword = ref(false);

const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};

const handleLogin = async () => {
  const { loginApi, getCorrentUser } = authAPI();

  try {
    const response = await loginApi(loginform);
    if (response) {
      authStore.addToken(response?.data?.access_token);
      const get_user = await getCorrentUser(response.data.access_token);

      if (get_user) {
        authStore.setUserData(get_user.data);
        useNuxtApp().$toast('Logged in Successfully');

        switch (get_user.data.user_type) {
          case UserType.SUPERADMIN:
          case UserType.ADMIN:
            navigateTo('/admin', { redirectCode: 301 });
            break;
          case UserType.VENDOR:
            navigateTo('/vendor', { redirectCode: 301 });
            break;
          default:
            navigateTo('/user', { redirectCode: 301 });
            break;
        }
      }
    } else {
      toast.error('Invalid credentials', {
        position: 'top-right',
        autoClose: 3000,
      });
    }
  } catch (error) {
    toast.error('Login failed, please try again later', {
      position: 'top-right',
      autoClose: 3000,
    });

    console.error('Login failed', error);
  }
};

watch(authStore.isAuthenticated, (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`);
})

</script>

<style scoped>
/* Import Font Awesome icons */
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css');
</style>

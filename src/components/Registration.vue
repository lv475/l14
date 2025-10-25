<script setup>
import { reactive, computed, ref } from 'vue'

const form = reactive({
  name: '',
  email: '',
  age: null,
  accountType: '',
  password: '',
  password2: '',
  agree: false
})

const touched = reactive({
  name: false,
  email: false,
  age: false,
  accountType: false,
  password: false,
  password2: false,
  agree: false
})

const emailBusy = ref(false)
let emailTimeout = null

const reEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

const isNameValid = computed(() => form.name.trim().length >= 2)
const isEmailValid = computed(() => reEmail.test(form.email))
const isAgeValid = computed(() => typeof form.age === 'number' && form.age >= 14 && form.age <= 120)
const isAccountTypeValid = computed(() => form.accountType !== '')
const isPasswordStrong = computed(() => form.password.length >= 8 && /\d/.test(form.password))
const isPasswordMatch = computed(() => form.password === form.password2)
const isAgree = computed(() => form.agree === true)

const errors = reactive({
  name: '',
  email: '',
  age: '',
  accountType: '',
  password: '',
  password2: '',
  agree: ''
})

const isValid = computed(() => 
  Object.values(errors).every(error => error === '') &&
  !emailBusy.value
)

function validateField(field) {
  switch (field) {
    case 'name':
      errors.name = form.name.trim().length >= 2 ? '' : 'Имя должно содержать от 2 до 50 символов'
      break
    case 'email':
      if (!form.email.trim()) {
        errors.email = 'Укажите email'
      } else if (!reEmail.test(form.email)) {
        errors.email = 'Неверный формат email'
      }
      break
    case 'age':
      if (form.age === null || form.age === '') {
        errors.age = 'Укажите возраст'
      } else if (!isAgeValid.value) {
        errors.age = 'Возраст должен быть от 14 до 120 лет'
      } else {
        errors.age = ''
      }
      break
    case 'accountType':
      errors.accountType = isAccountTypeValid.value ? '' : 'Выберите тип аккаунта'
      break
    case 'password':
      if (!form.password) {
        errors.password = 'Укажите пароль'
      } else if (!isPasswordStrong.value) {
        errors.password = 'Пароль должен содержать минимум 8 символов и хотя бы одну цифру'
      } else {
        errors.password = ''
      }
      break
    case 'password2':
      errors.password2 = isPasswordMatch.value ? '' : 'Пароли не совпадают'
      break
    case 'agree':
      errors.agree = isAgree.value ? '' : 'Необходимо согласие с условиями'
      break
  }
}

function checkEmailAvailability(email) {
  emailBusy.value = true

  setTimeout(() => {
    if (email.endsWith('@test.com')) {
      errors.email = 'Этот email уже занят'
    } else if (reEmail.test(email)) {
      errors.email = '' 
    }
    emailBusy.value = false
  }, 500)
}

function handleEmailInput() {
  if (emailTimeout) {
    clearTimeout(emailTimeout)
  }
  
  validateField('email')

  emailTimeout = setTimeout(() => {
    if (form.email && !errors.email) {
      checkEmailAvailability(form.email)
    }
  }, 500)
}

function onBlur(field) {
  touched[field] = true
  validateField(field)
}

function inputClass(field) {
  if (!touched[field]) return ''
  return errors[field] ? 'is-invalid' : 'is-valid'
}

function onSubmit(e) {
  e.preventDefault()
  Object.keys(touched).forEach(k => {
    touched[k] = true
    validateField(k)
  })
  
  if (!isValid.value) return

  alert('Форма валидна! Данные готовы к отправке в API.')
  console.log('Данные формы:', form)
}
</script>

<template>
  <form @submit="onSubmit" novalidate class="form">
    <label>Имя</label>
    <input
      type="text"
      v-model.trim="form.name"
      @blur="onBlur('name')"
      :class="inputClass('name')"
      required
      minlength="2"
      maxlength="50"
      placeholder="Иван"
    />
    <p v-if="touched.name && errors.name" class="error">{{ errors.name }}</p>

    <label>Email</label>
    <input
      type="email"
      v-model.trim="form.email"
      @blur="onBlur('email')"
      @input="handleEmailInput"
      :class="inputClass('email')"
      required
      placeholder="user@example.com"
    />
    <p v-if="emailBusy">Проверяем email...</p>
    <p v-if="touched.email && errors.email" class="error">{{ errors.email }}</p>

    <label>Возраст</label>
    <input
      type="number"
      v-model.number="form.age"
      @blur="onBlur('age')"
      :class="inputClass('age')"
      required
      min="14"
      max="120"
      placeholder="например, 18"
    />
    <small>Тип данных: {{ typeof form.age }}</small>
    <p v-if="touched.age && errors.age" class="error">{{ errors.age }}</p>

    <label>Тип аккаунта</label>
    <div class="radio-group">
      <label>
        <input 
          type="radio" 
          value="student" 
          v-model="form.accountType" 
          @blur="onBlur('accountType')" 
        />
        Студент
      </label>
      <label>
        <input 
          type="radio" 
          value="business" 
          v-model="form.accountType" 
          @blur="onBlur('accountType')" 
        />
        Бизнес
      </label>
    </div>
    <p v-if="touched.accountType && errors.accountType" class="error">{{ errors.accountType }}</p>

    <label>Пароль</label>
    <input
      type="password"
      v-model.lazy="form.password"
      @blur="onBlur('password')"
      :class="inputClass('password')"
      required
      minlength="8"
      placeholder="минимум 8 символов, включая цифру"
    />
    <p v-if="touched.password && errors.password" class="error">{{ errors.password }}</p>

    <label>Подтверждение пароля</label>
    <input
      type="password"
      v-model.lazy="form.password2"
      @blur="onBlur('password2')"
      :class="inputClass('password2')"
      required
      minlength="8"
      placeholder="повторите пароль"
    />
    <p v-if="touched.password2 && errors.password2" class="error">{{ errors.password2 }}</p>

    <label class="agree">
      <input type="checkbox" v-model="form.agree" @blur="onBlur('agree')" />
      Я согласен с правилами
    </label>
    <p v-if="touched.agree && errors.agree" class="error">{{ errors.agree }}</p>

    <button type="submit">Отправить</button>

    <pre class="debug">{{ form }}</pre>
  </form>
</template>

<style scoped>
.form { max-width: 520px; display: grid; gap: .5rem; }
input, select, textarea { padding: .5rem; border: 1px solid #cbd5e1; border-radius: .5rem; }
.is-invalid { border-color: #e74444; outline-color: #e74444; }
.is-valid { border-color: #22c55e; outline-color: #22c55e; }
.error { color: #b91c1c; font-size: .9rem; }
.agree { display: flex; gap: .5rem; align-items: center; }
.debug { background: #bf172a; color: #e2e8f0; padding: .5rem; border-radius: .5rem; }
.radio-group { display: flex; gap: 1rem; }
.radio-group label { display: flex; gap: 0.5rem; align-items: center; }
</style>
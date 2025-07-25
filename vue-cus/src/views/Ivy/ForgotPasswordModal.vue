<template>
    <!-- 有改過 -->
    <div class="modal-bg" v-if="show" @click="emit('close')">
        <div class="modal-dialog modal-dialog-centered" @click.stop>
            <div class="modal-content p-4">
                <div class="modal-header border-0 pb-0 justify-content-between">
                    <button class="btn nav-btn" @click="emit('back')">
                        <svg width="28" height="28" fill="none" viewBox="0 0 24 24">
                            <path d="M15 6l-6 6 6 6" stroke="#222" stroke-width="2" stroke-linecap="round"
                                stroke-linejoin="round" />
                        </svg>
                    </button>
                    <button type="button" class="btn-close custom-close" @click="emit('close')"></button>
                </div>
                <div class="modal-body d-flex flex-column align-items-center pt-0">
                    <img src="https://cdn-icons-png.flaticon.com/512/1745/1745318.png" alt="lock" class="icon-img mb-3">
                    <div class="fw-bold title mb-2">忘記密碼了嗎？</div>
                    <div class="desc mb-3">輸入你的email，我們發送重設密碼連結</div>
                    <form class="w-100" @submit.prevent="onSubmit">
                        <div class="mb-3 text-start w-100">
                            <label class="form-label label-strong">電子郵件</label>
                            <input
                                type="email"
                                v-model="email"
                                class="form-control custom-input"
                                required
                                placeholder="eattiy@msn.com"
                            />
                        </div>
                        <button type="submit" class="btn btn-main w-100 mb-1" :disabled="loading">
                            {{ loading ? '寄送中...' : '重設密碼' }}
                        </button>
                    </form>
                    <button class="btn-link mt-2" @click="emit('back')">回到登入頁</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import { useUserStore } from '@/stores/user.js'
import axios from '@/plungins/axios.js'

const props = defineProps({ show: Boolean })
const emit = defineEmits(['close', 'back', 'submit'])

const userStore = useUserStore()
const loading = ref(false)

const email = computed({
    get: () => userStore.email,
    set: v => userStore.setEmail(v)
})

async function onSubmit() {
    if (!email.value) {
        Swal.fire({
            icon: 'warning', // 顯示一個黃色的警告圖示
            title: '請注意', // 彈出視窗的標題
            text: '請輸入您的 Email 地址。', // 提示的具體內容
            confirmButtonText: '確定' // 確認按鈕的文字
        });
        return
    }
    loading.value = true
    try {
        // 必須用 URLSearchParams，對應 @RequestParam
        const params = new URLSearchParams()
        params.append('email', email.value.trim())

        await axios.post('/api/send-reset-password', params)
        Swal.fire({
            icon: 'success', // 顯示成功圖示（綠色勾勾）
            title: '重設密碼連結已寄出！', // 標題
            html: `我們已將重設密碼的連結寄送到 <b>${email.value}</b>。<br>請前往您的信箱查收，並點擊連結以完成密碼重設。`, // 內容支援 HTML
            confirmButtonText: '確定' // 確認按鈕文字
        });
        emit('submit', email.value)
        // 這裡也可以 emit('close') 或跳出"已寄出"的 modal
    } catch (e) {
        const errorMessage = e.response?.data?.message || '寄送失敗，請稍後再試';
        Swal.fire({
            icon: 'error', // 顯示錯誤圖示
            title: '操作失敗', // 標題
            text: errorMessage, // 內容文字，優先顯示後端訊息，否則顯示預設訊息
            confirmButtonText: '確定' // 確認按鈕文字
        });
        
    } finally {
        loading.value = false
    }
}
</script>

<style scoped>
.modal-bg {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.08);
    z-index: 900;
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-dialog {
    width: 400px;
    /* margin: 0 auto; */
}

.modal-content {
    background: #fff !important;
    border-radius: 20px;
    box-shadow: 0 2px 24px 4px rgba(0, 0, 0, 0.10);
    border: none;
    padding: 2.2rem 2rem 2rem 2rem;
    position: relative;
}

.close-btn {
    position: absolute;
    top: 14px;
    right: 14px;
    background: #fff;
    border: none;
    border-radius: 50%;
    box-shadow: 0 2px 8px 1px rgba(0, 0, 0, 0.10);
    width: 40px;
    height: 40px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    z-index: 10;
    transition: background 0.15s;
}

.nav-btn {
    background: none;
    border: none;
    padding: 0;
    margin-left: -5px;
    margin-top: -5px;
    box-shadow: none;
    outline: none;
}

.icon-img {
    width: 62px;
    height: 62px;
    object-fit: contain;
    margin-bottom: 8px;
}

.title {
    font-size: 2rem;
    font-weight: bold;
    color: #222;
}

.desc {
    color: #999;
    font-size: 1.1rem;
    font-weight: 400;
}

.label-strong {
    font-weight: bold;
    color: #222;
    font-size: 1.08rem;
}

.custom-input {
    font-size: 1.1rem;
    border: 2px solid #222;
    border-radius: 8px;
    height: 46px;
    padding: 7px 12px;
    margin-top: 5px;
    background: #fff;
    box-shadow: none;
}

.custom-input:focus {
    border-color: #ffba20;
    box-shadow: 0 0 0 1px #ffba2021;
}

.btn-main {
    background: #ffba20;
    color: #fff;
    font-weight: bold;
    font-size: 20px;
    height: 48px;
    border-radius: 10px;
    border: none;
    letter-spacing: 2px;
    transition: filter 0.15s;
    box-shadow: 0 2px 8px 1px #ffba200f;
}

.btn-main:hover {
    filter: brightness(1.12);
    background: #f1cd78;
}

.btn-link {
    color: #ffba20;
    background: none;
    border: none;
    font-weight: bold;
    font-size: 1rem;
    text-decoration: underline;
    cursor: pointer;
    padding: 0;
    margin: 0;
}
</style>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useSessionStore } from '../stores/session';
import { useToastStore } from '../stores/toast';
import { api } from '../lib/http';
import { storeToRefs } from 'pinia';
import BrandLogo from '../components/layout/BrandLogo.vue';

const router = useRouter();
const sessionStore = useSessionStore();
const { sessionState, publicConfig } = storeToRefs(sessionStore);
const { showToast } = useToastStore();

const config = ref({});
const heroConfig = computed(() => config.value.hero || {
    title1: '发现',
    title2: '优质订阅',
    description: '浏览并获取由管理员分享的精选订阅组合，一键导入到您的客户端。'
});
const announcement = computed(() => config.value.announcement);
const guestbookConfig = computed(() => config.value.guestbook || {});
const profileCount = ref(0);
const nodeCount = ref(0);
const loading = ref(true);

const stats = ref([
    { label: '订阅组', value: '0', icon: 'M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10', color: 'from-violet-500 to-purple-600' },
    { label: '代理节点', value: '0', icon: 'M8 7h12m0 0l-4-4m4 4l-4 4m0 6H4m0 0l4 4m-4-4l4-4', color: 'from-indigo-500 to-blue-600' },
    { label: '支持协议', value: '11+', icon: 'M13 10V3L4 14h7v7l9-11h-7z', color: 'from-purple-500 to-pink-600' },
    { label: '客户端兼容', value: '7+', icon: 'M9.75 17L9 20l-1 1h8l-1-1-.75-3M3 13h18M5 17h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z', color: 'from-cyan-500 to-blue-600' },
]);

const fetchConfig = async () => {
    try {
        loading.value = true;
        const data = await api.get('/api/public/profiles');
        if (data.success) {
            config.value = data.config || {};
            profileCount.value = (data.data || []).length;

            let totalNodes = 0;
            (data.data || []).forEach(profile => {
                if (profile.nodeCount) {
                    totalNodes += profile.nodeCount;
                }
            });
            nodeCount.value = totalNodes;

            stats.value[0].value = String(profileCount.value);
            stats.value[1].value = String(nodeCount.value);
        }
    } catch (e) {
        console.error('Failed to fetch landing config', e);
    } finally {
        loading.value = false;
    }
};

const isLoggedIn = computed(() => sessionState.value === 'loggedIn');
const isPublicPageEnabled = computed(() => publicConfig.value?.enablePublicPage !== false);

const goToLogin = () => {
    const rawPath = sessionStore.publicConfig?.customLoginPath;
    const normalizedPath = (rawPath && typeof rawPath === 'string') ? rawPath.trim().replace(/^\/+/, '') : '';
    if (normalizedPath && normalizedPath !== 'login') {
        router.push(`/${normalizedPath}`);
    } else {
        router.push('/login');
    }
};

const goToExplore = () => {
    router.push('/explore');
};

const goToDashboard = () => {
    router.push('/dashboard');
};

onMounted(() => {
    fetchConfig();
});
</script>

<template>
    <div class="min-h-screen bg-cosmic-dust bg-frost-pattern transition-colors duration-500 selection:bg-primary-500/30 selection:text-white relative">

        <!-- Navigation -->
        <header class="sticky top-0 z-50 backdrop-blur-2xl bg-white/75 dark:bg-[#030712]/75 border-b border-gray-200/20 dark:border-white/5 transition-all duration-300">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="flex justify-between items-center h-16 md:h-[72px]">
                    <BrandLogo text-size-class="text-lg" :icon-size="34" />
                    <div class="flex items-center gap-3">
                        <button
                            v-if="isPublicPageEnabled && !isLoggedIn"
                            @click="goToExplore"
                            class="hidden sm:inline-flex items-center gap-2 px-4 py-2 rounded-full text-sm font-medium text-gray-600 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white hover:bg-gray-100 dark:hover:bg-white/10 transition-all duration-200"
                        >
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M4 6a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2V6zM14 6a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2V6zM4 16a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2v-2zM14 16a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2v-2z" />
                            </svg>
                            浏览订阅
                        </button>
                        <button
                            v-if="isLoggedIn"
                            @click="goToDashboard"
                            class="inline-flex items-center gap-2 px-5 py-2.5 rounded-full text-sm font-semibold text-white bg-primary-600 hover:bg-primary-700 shadow-lg shadow-primary-500/25 hover:shadow-primary-500/40 transition-all duration-300 active:scale-95"
                        >
                            进入仪表盘
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M13 7l5 5m0 0l-5 5m5-5H6" />
                            </svg>
                        </button>
                        <button
                            v-else
                            @click="goToLogin"
                            class="inline-flex items-center gap-2 px-5 py-2.5 rounded-full text-sm font-semibold text-white bg-primary-600 hover:bg-primary-700 shadow-lg shadow-primary-500/25 hover:shadow-primary-500/40 transition-all duration-300 active:scale-95"
                        >
                            管理员登录
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                            </svg>
                        </button>
                    </div>
                </div>
            </div>
        </header>

        <!-- Hero Section -->
        <section class="relative pt-16 pb-12 lg:pt-28 lg:pb-20 z-10 overflow-hidden">
            <!-- Large decorative blobs -->
            <div class="absolute inset-0 overflow-hidden pointer-events-none" aria-hidden="true">
                <div class="absolute -top-40 -right-40 w-[600px] h-[600px] rounded-full bg-gradient-to-br from-primary-400/10 to-purple-500/5 dark:from-primary-500/15 dark:to-purple-600/8 blur-3xl"></div>
                <div class="absolute -bottom-60 -left-40 w-[500px] h-[500px] rounded-full bg-gradient-to-tr from-indigo-400/8 to-cyan-400/5 dark:from-indigo-500/12 dark:to-cyan-500/6 blur-3xl"></div>
                <div class="absolute top-1/3 left-1/2 -translate-x-1/2 w-[800px] h-[400px] rounded-full bg-gradient-to-r from-violet-400/5 via-purple-400/5 to-indigo-400/5 dark:from-violet-500/8 dark:via-purple-500/8 dark:to-indigo-500/8 blur-3xl"></div>
            </div>

            <!-- Floating dots -->
            <div class="absolute inset-0 overflow-hidden pointer-events-none" aria-hidden="true">
                <div class="absolute top-24 left-[8%] w-2 h-2 rounded-full bg-primary-300/50 dark:bg-primary-500/40 animate-float-slow"></div>
                <div class="absolute top-36 left-[15%] w-1.5 h-1.5 rounded-full bg-purple-300/60 dark:bg-purple-500/45 animate-float" style="animation-delay: 0.8s;"></div>
                <div class="absolute top-48 right-[10%] w-3 h-3 rounded-full bg-primary-400/30 dark:bg-primary-400/25 animate-float-slow" style="animation-delay: 1.5s; animation-duration: 7s;"></div>
                <div class="absolute top-28 right-[22%] w-1.5 h-1.5 rounded-full bg-indigo-300/50 dark:bg-indigo-500/35 animate-float" style="animation-delay: 2s; animation-duration: 5s;"></div>
                <div class="absolute bottom-32 left-[20%] w-2.5 h-2.5 rounded-full bg-purple-300/40 dark:bg-purple-500/30 animate-float-slow" style="animation-delay: 3s;"></div>
            </div>

            <div class="max-w-7xl mx-auto px-6 sm:px-8 lg:px-12 relative z-10">
                <div class="text-center max-w-4xl mx-auto">
                    <!-- Badge -->
                    <div class="hero-badge mx-auto mb-8" style="animation-delay: 0ms;">
                        <span class="hero-badge-dot">
                            <span class="hero-badge-dot-ping"></span>
                            <span class="hero-badge-dot-core"></span>
                        </span>
                        <span class="text-xs font-bold text-primary-700 dark:text-primary-300 tracking-widest uppercase">Premium Subscription Service</span>
                    </div>

                    <!-- Title -->
                    <h1 class="text-4xl sm:text-6xl lg:text-7xl xl:text-8xl font-black tracking-tight leading-[1.05] mb-6 animate-fade-in-up" style="animation-delay: 50ms;">
                        <span class="block text-gray-900 dark:text-white">{{ heroConfig.title1 }}</span>
                        <span class="block hero-gradient-text mt-3 pb-3">
                            {{ heroConfig.title2 }}
                        </span>
                    </h1>

                    <!-- Description -->
                    <p class="text-base sm:text-lg lg:text-xl text-gray-500 dark:text-gray-400 leading-relaxed font-medium max-w-3xl mx-auto mb-10 animate-fade-in-up" style="animation-delay: 150ms;">
                        {{ heroConfig.description }}
                    </p>

                    <!-- CTA Buttons -->
                    <div class="flex flex-col sm:flex-row items-center justify-center gap-4 animate-fade-in-up" style="animation-delay: 250ms;">
                        <button
                            @click="goToExplore"
                            class="btn-glow inline-flex items-center gap-3 px-8 py-3.5 rounded-2xl text-base font-semibold text-white bg-primary-600 hover:bg-primary-700 shadow-xl shadow-primary-500/25 hover:shadow-primary-500/40 transition-all duration-300 active:scale-95"
                        >
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                            </svg>
                            开始浏览
                        </button>
                        <button
                            @click="goToLogin"
                            class="inline-flex items-center gap-3 px-8 py-3.5 rounded-2xl text-base font-semibold text-gray-700 dark:text-gray-300 bg-white/60 dark:bg-white/5 backdrop-blur-xl border border-gray-200/60 dark:border-white/10 hover:border-primary-300/50 dark:hover:border-primary-500/30 hover:bg-white/80 dark:hover:bg-white/10 shadow-sm hover:shadow-md transition-all duration-300 active:scale-95"
                        >
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                            </svg>
                            管理员登录
                        </button>
                    </div>
                </div>

                <!-- Stats Cards -->
                <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 sm:gap-6 mt-16 lg:mt-20 max-w-4xl mx-auto animate-fade-in-up" style="animation-delay: 350ms;">
                    <div v-for="stat in stats" :key="stat.label"
                        class="group relative glass-radiant rounded-2xl p-5 sm:p-6 text-center transition-all duration-400"
                    >
                        <div class="inline-flex items-center justify-center w-10 h-10 sm:w-12 sm:h-12 rounded-xl mb-3"
                            :class="`bg-gradient-to-br ${stat.color} text-white shadow-lg group-hover:scale-110 transition-transform duration-300`"
                        >
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 sm:w-6 sm:h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                <path stroke-linecap="round" stroke-linejoin="round" :d="stat.icon" />
                            </svg>
                        </div>
                        <div class="text-2xl sm:text-3xl font-extrabold text-gray-900 dark:text-white mb-1">{{ stat.value }}</div>
                        <div class="text-xs sm:text-sm text-gray-500 dark:text-gray-400 font-medium">{{ stat.label }}</div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Features Section -->
        <section class="relative z-10 py-16 lg:py-24">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="text-center mb-16">
                    <span class="section-eyebrow mb-4">Why Choose Us</span>
                    <h2 class="text-3xl md:text-4xl font-extrabold text-gray-900 dark:text-white tracking-tight mt-3">
                        强大功能
                    </h2>
                    <p class="mt-4 text-gray-500 dark:text-gray-400 max-w-2xl mx-auto">
                        一站式订阅管理解决方案，支持多协议、多平台、多客户端
                    </p>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 lg:gap-8 max-w-5xl mx-auto">
                    <div v-for="(feature, idx) in [
                        { title: '多协议支持', desc: '支持 Shadowsocks, VMess, VLESS, Trojan, Hysteria2, TUIC, Snell 等主流协议', icon: 'M12 11c0 3.517-1.009 6.799-2.753 9.571m-3.44-2.04l.054-.09A13.916 13.916 0 008 11a4 4 0 118 0c0 1.017-.07 2.019-.203 3m-2.118 6.844A21.88 21.88 0 0015.171 17m3.839 1.132c.645-2.266.99-4.659.99-7.132A8 8 0 008 4.07M3 15.364c.64-1.319 1-2.8 1-4.364 0-1.457.39-2.823 1.07-4' },
                        { title: '多平台兼容', desc: '兼容 Clash, Surge, Loon, Quantumult X, Stash, Shadowrocket 等主流客户端', icon: 'M9.75 17L9 20l-1 1h8l-1-1-.75-3M3 13h18M5 17h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z' },
                        { title: '智能节点管理', desc: '自动排序、去重、重命名、过滤节点，支持可视化算子链配置', icon: 'M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15' },
                        { title: '一键导入', desc: '生成订阅链接一键导入客户端，支持二维码扫码订阅', icon: 'M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z' },
                        { title: '自动同步', desc: '定时自动更新订阅，保持节点信息始终最新，无需手动操作', icon: 'M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15' },
                        { title: '流量统计', desc: '实时查看剩余流量和到期时间，总览所有订阅状态', icon: 'M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z' },
                    ]" :key="idx"
                        :style="`animation-delay: ${400 + idx * 80}ms;`"
                        class="group glass-radiant rounded-2xl p-6 sm:p-8 text-left animate-fade-in-up transition-all duration-400"
                    >
                        <div class="inline-flex items-center justify-center w-12 h-12 rounded-xl bg-primary-50 dark:bg-primary-500/10 text-primary-600 dark:text-primary-400 mb-5 group-hover:scale-110 group-hover:bg-primary-100 dark:group-hover:bg-primary-500/20 transition-all duration-300">
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
                                <path stroke-linecap="round" stroke-linejoin="round" :d="feature.icon" />
                            </svg>
                        </div>
                        <h3 class="text-lg font-bold text-gray-900 dark:text-white mb-2">{{ feature.title }}</h3>
                        <p class="text-sm text-gray-500 dark:text-gray-400 leading-relaxed">{{ feature.desc }}</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- CTA Section -->
        <section class="relative z-10 py-16 lg:py-20">
            <div class="max-w-3xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
                <div class="glass-radiant rounded-3xl p-10 sm:p-14 lg:p-16 animate-fade-in-up">
                    <h2 class="text-2xl sm:text-3xl lg:text-4xl font-extrabold text-gray-900 dark:text-white mb-4">
                        准备好开始了吗？
                    </h2>
                    <p class="text-gray-500 dark:text-gray-400 mb-8 max-w-lg mx-auto">
                        立即浏览订阅组，或登录管理面板进行配置
                    </p>
                    <div class="flex flex-col sm:flex-row items-center justify-center gap-4">
                        <button
                            @click="goToExplore"
                            class="btn-glow inline-flex items-center gap-3 px-8 py-3.5 rounded-2xl text-base font-semibold text-white bg-primary-600 hover:bg-primary-700 shadow-xl shadow-primary-500/25 transition-all duration-300 active:scale-95"
                        >
                            浏览订阅
                            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M13 7l5 5m0 0l-5 5m5-5H6" />
                            </svg>
                        </button>
                        <button
                            @click="goToLogin"
                            class="inline-flex items-center gap-3 px-8 py-3.5 rounded-2xl text-base font-semibold text-gray-700 dark:text-gray-300 bg-white/60 dark:bg-white/5 backdrop-blur-xl border border-gray-200/60 dark:border-white/10 hover:border-primary-300/50 dark:hover:border-primary-500/30 shadow-sm hover:shadow-md transition-all duration-300 active:scale-95"
                        >
                            管理面板
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- Footer -->
        <footer class="relative z-10 border-t border-gray-200/30 dark:border-white/5">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
                <div class="flex flex-col sm:flex-row items-center justify-between gap-4">
                    <div class="flex items-center gap-2 text-sm text-gray-400 dark:text-gray-500">
                        <BrandLogo text-size-class="text-sm" :icon-size="20" />
                    </div>
                    <div class="flex items-center gap-6">
                        <a href="https://github.com/imzyb/MiSub" target="_blank" rel="noopener noreferrer"
                            class="text-sm text-gray-400 dark:text-gray-500 hover:text-gray-600 dark:hover:text-gray-300 transition-colors">
                            GitHub
                        </a>
                        <button
                            v-if="guestbookConfig?.enabled !== false"
                            @click="$emit('open-guestbook')"
                            class="text-sm text-gray-400 dark:text-gray-500 hover:text-gray-600 dark:hover:text-gray-300 transition-colors"
                        >
                            留言板
                        </button>
                        <span class="text-sm text-gray-400 dark:text-gray-500">
                            &copy; {{ new Date().getFullYear() }}
                        </span>
                    </div>
                </div>
            </div>
        </footer>
    </div>
</template>

<style scoped>
.hero-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    border-radius: 9999px;
    background: rgba(255, 255, 255, 0.55);
    backdrop-filter: blur(24px) saturate(180%);
    -webkit-backdrop-filter: blur(24px) saturate(180%);
    border: 1px solid rgba(255, 255, 255, 0.6);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.04), inset 0 1px 0 rgba(255, 255, 255, 0.6);
    animation: fadeInUp 0.8s cubic-bezier(0.16, 1, 0.3, 1) forwards;
    opacity: 0;
}

.dark .hero-badge {
    background: rgba(15, 23, 42, 0.65);
    border-color: rgba(255, 255, 255, 0.08);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

.hero-badge-dot {
    position: relative;
    display: flex;
    width: 0.5rem;
    height: 0.5rem;
}

.hero-badge-dot-ping {
    position: absolute;
    display: inline-flex;
    width: 100%;
    height: 100%;
    border-radius: 9999px;
    background-color: rgb(167, 139, 250);
    opacity: 0.75;
    animation: ping 1.5s cubic-bezier(0, 0, 0.2, 1) infinite;
}

.hero-badge-dot-core {
    position: relative;
    display: inline-flex;
    width: 0.5rem;
    height: 0.5rem;
    border-radius: 9999px;
    background-color: rgb(139, 92, 246);
}

@keyframes fadeInUp {
    from { opacity: 0; transform: translateY(20px); }
    to { opacity: 1; transform: translateY(0); }
}

@keyframes ping {
    75%, 100% { transform: scale(2); opacity: 0; }
}

.animate-fade-in-up {
    animation: fadeInUp 0.7s cubic-bezier(0.16, 1, 0.3, 1) forwards;
    opacity: 0;
}

@media (prefers-reduced-motion: reduce) {
    .animate-fade-in-up {
        animation: none !important;
        opacity: 1;
    }
    .hero-badge {
        animation: none !important;
        opacity: 1;
    }
}
</style>

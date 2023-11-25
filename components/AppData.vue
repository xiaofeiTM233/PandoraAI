<script setup>
import { storeToRefs } from 'pinia';

const conversationsStore = useConversationsStore();
const {
    processingController,
} = storeToRefs(conversationsStore);

const importAppData = () => {
    if (processingController.value) {
        return;
    }
    // 从用户获取 JSON 文件，并将其转换为 localStorage
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = 'application/json';
    input.onchange = (e) => {
        if (processingController.value) {
            return;
        }
        const file = e.target.files[0];
        const reader = new FileReader();
        reader.readAsText(file, 'UTF-8');
        reader.onload = (readerEvent) => {
            if (processingController.value) {
                return;
            }
            const content = readerEvent.target.result;
            localStorage.clear();
            Object.assign(localStorage, JSON.parse(content));
            window.location.reload();
        };
    };
    input.click();
};

const exportAppData = () => {
    if (processingController.value) {
        return;
    }
    // 将 localStorage 转换为 JSON 并另存为为文件
    const dataStr = JSON.stringify(localStorage);
    const dataUri = `data:application/json;charset=utf-8,${encodeURIComponent(dataStr)}`;

    const exportFileDefaultName = `PandoraAI-${Date.now()}.data.json`;

    const linkElement = document.createElement('a');
    linkElement.setAttribute('href', dataUri);
    linkElement.setAttribute('download', exportFileDefaultName);
    linkElement.click();
};
</script>

<template>
    <button
        @click="importAppData"
        class="flex items-center gap-1 text-white/60 transition duration-300 hover:text-white/80"
        :disabled="!!processingController"
        :class="{ 'cursor-not-allowed': !!processingController }"
    >
        <Icon class="w-4 h-4" name="bx:bx-import" />
        导入
    </button>
    <button
        @click="exportAppData"
        class="flex items-center gap-1 text-white/60 transition duration-300 hover:text-white/80"
        :disabled="!!processingController"
        :class="{ 'cursor-not-allowed': !!processingController }"
    >
        <Icon class="w-4 h-4" name="bx:bx-export" />
        导出
    </button>
</template>

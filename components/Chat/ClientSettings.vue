<script setup>
import {
    Dialog,
    DialogTitle,
    DialogDescription,
    DialogPanel,
    Switch,
} from '@headlessui/vue';
import get from 'lodash/get';
import set from 'lodash/set';
import unset from 'lodash/unset';
import BingIcon from '~/components/Icons/BingIcon.vue';
import GPTIcon from '~/components/Icons/GPTIcon.vue';

const props = defineProps({
    isOpen: {
        type: Boolean,
        required: true,
    },
    setIsOpen: {
        type: Function,
        required: true,
    },
    client: {
        type: String,
    },
    presetName: {
        type: String,
    },
});

const availableOptions = {
    chatgpt: {
        stream: {
            type: 'checkbox',
            label: '流式传输',
            default: true,
        },
        shouldGenerateTitle: {
            type: 'checkbox',
            label: '自动生成标题',
            default: true,
        },
        clientOptions: {
            type: 'nested',
            label: '客户端设置',
            properties: {
                openaiApiKey: {
                    type: 'text',
                    label: 'OpenAI API Key',
                },
                reverseProxyUrl: {
                    type: 'text',
                    label: '反代 URL',
                },
                maxContextTokens: {
                    type: 'number',
                    label: '最大上下文令牌',
                    min: 1,
                },
                maxPromptTokens: {
                    type: 'number',
                    label: '最大提示令牌',
                    min: 1,
                },
                userLabel: {
                    type: 'text',
                    label: "User名称",
                },
                chatGptLabel: {
                    type: 'text',
                    label: "AI名称",
                },
                promptPrefix: {
                    type: 'textarea',
                    label: '使用说明（提示前缀）',
                },
                modelOptions: {
                    type: 'nested',
                    label: '模型设置',
                    properties: {
                        model: {
                            type: 'text',
                            label: '模型',
                        },
                        temperature: {
                            type: 'range',
                            label: 'Temperature',
                            min: 0,
                            max: 2,
                            step: 0.01,
                        },
                        top_p: {
                            type: 'range',
                            label: 'Top P',
                            min: 0,
                            max: 2,
                            step: 0.01,
                        },
                        presence_penalty: {
                            type: 'range',
                            label: 'Presence Penalty',
                            min: -2,
                            max: 2,
                            step: 0.01,
                        },
                        frequency_penalty: {
                            type: 'range',
                            label: 'Frequency Penalty',
                            min: -2,
                            max: 2,
                            step: 0.01,
                        },
                        max_tokens: {
                            type: 'range',
                            label: 'Max Tokens',
                            min: 1,
                            max: 4096,
                            step: 1,
                        },
                    },
                },
            },
        },
    },
    'chatgpt-browser': {
        stream: {
            type: 'checkbox',
            label: '流式传输',
            default: true,
        },
        clientOptions: {
            type: 'nested',
            label: '客户端设置',
            properties: {
                reverseProxyUrl: {
                    type: 'text',
                    label: '反代 URL',
                },
                accessToken: {
                    type: 'textarea',
                    label: '访问令牌',
                },
                cookies: {
                    type: 'textarea',
                    label: 'Cookies',
                },
            },
        },
    },
    bing: {
        stream: {
            type: 'checkbox',
            label: '流式传输',
            default: true,
        },
        jailbreakMode: {
            type: 'checkbox',
            label: '越狱模式',
        },
        toneStyle: {
            type: 'select',
            label: '语气风格',
            options: [
                {
                    label: '创意',
                    value: 'creative',
                },
                {
                    label: '平衡',
                    value: 'balanced',
                },
                {
                    label: '精确',
                    value: 'precise',
                },
            ],
        },
        clientOptions: {
            type: 'nested',
            label: '客户端设置',
            properties: {
                host: {
                    type: 'text',
                    label: 'Host (例如: https://cn.bing.com)',
                },
                cookies: {
                    type: 'textarea',
                    label: 'Cookies',
                },
            },
        },
    },
};

const presetStore = usePresetsStore();
const {
    setPreset,
    getPreset,
    deletePreset,
    setActivePresetName,
} = presetStore;

// 根据客户端使用 'switch' 选择默认 'saveAsName'
const defaultSaveAsName = computed(() => {
    if (!props.client) {
        return '';
    }
    if (props.presetName !== props.client) {
        return props.presetName;
    }
    switch (props.client) {
        case 'chatgpt':
            return 'OpenAI API';
        case 'chatgpt-browser':
            return 'ChatGPT';
        case 'bing':
            return 'Bing';
        default:
            throw new Error('无效的客户端');
    }
});

const saveAsName = ref(defaultSaveAsName.value);

const formClientOptions = ref({});

// 递归表单生成组件
const generateForm = (options, parentKey, levels = 0) => Object.entries(options).map(([key]) => {
    const option = options[key];
    let optionKey;
    if (key === '0') {
        optionKey = parentKey;
    } else {
        optionKey = `${parentKey}.${key}`;
    }
    if (option.type === 'nested') {
        return h('div', { class: 'flex flex-col gap-2', style: `margin-left: ${levels}em;` }, [
            h('label', { class: 'text-white/80 font-bold' }, option.label),
            ...generateForm(option.properties, optionKey, levels + 1),
        ]);
    }
    // 其他类型，如文本，范围，复选框等。
    let classList = 'w-full placeholder-white/40 text-slate-300 text-sm rounded py-2 focus:outline-none';
    switch (option.type) {
        case 'range':
            classList = `${classList} bg-transparent`;
            break;
        default:
            classList = `${classList} shadow-inner bg-white/5 px-3`;
            break;
    }
    const inputValue = get(formClientOptions.value, optionKey, option.default);
    let inputElement;
    switch (option.type) {
        case 'textarea':
            inputElement = h('textarea', {
                placeholder: 'default server value',
                value: inputValue,
                onInput: (e) => {
                    const trimmedInputValue = e.target.value.trim();
                    if (!trimmedInputValue) {
                        unset(formClientOptions.value, optionKey);
                    } else {
                        set(formClientOptions.value, optionKey, trimmedInputValue);
                    }
                },
                class: classList,
            });
            break;
        case 'checkbox':
            inputElement = h(Switch, {
                modelValue: inputValue || false,
                'onUpdate:modelValue': (checked) => {
                    set(formClientOptions.value, optionKey, checked);
                },
                class: `relative inline-flex h-6 w-11 items-center rounded-full transition ${inputValue ? 'bg-white/50' : 'bg-white/10'}`,
            }, () => [
                h('span', { class: 'sr-only' }, option.label),
                h('span', {
                    class: `inline-block h-4 w-4 transform rounded-full bg-white transition ${inputValue ? 'translate-x-6' : 'translate-x-1'}`,
                }),
            ]);
            break;
        case 'select':
            inputElement = h('select', {
                value: inputValue,
                onChange: (e) => {
                    const targetValue = e.target.value;
                    if (!targetValue) {
                        unset(formClientOptions.value, optionKey);
                    } else {
                        set(formClientOptions.value, optionKey, targetValue);
                    }
                },
                class: classList,
            }, [
                h('option', { value: '' }, '服务器默认值'),
                ...option.options.map(_option => h('option', { value: _option.value }, _option.label)),
            ]);
            break;
        default:
            inputElement = h('input', {
                type: option.type,
                placeholder: '服务器默认值',
                value: inputValue,
                min: option.min,
                max: option.max,
                step: option.step || null,
                onInput: (e) => {
                    let eventInputValue = e.target.value;
                    if (typeof eventInputValue === 'undefined') {
                        unset(formClientOptions.value, optionKey);
                        return;
                    }
                    if (typeof eventInputValue === 'string') {
                        eventInputValue = eventInputValue.trim();
                    }
                    if (!eventInputValue) {
                        unset(formClientOptions.value, optionKey);
                        return;
                    }
                    if (option.type === 'number' || option.type === 'range') {
                        eventInputValue = Number(eventInputValue);
                    } else if (option.type === 'checkbox') {
                        eventInputValue = e.target.checked;
                    }
                    set(formClientOptions.value, optionKey, eventInputValue);
                },
                class: classList,
            });
            break;
    }

    return h('div', {
        class: 'flex flex-col gap-2',
    }, [
        h(
            'label',
            { class: 'text-white/60 text-xs' },
            option.type === 'range' ? `${option.label}: ${typeof inputValue === 'undefined' ? '服务器默认值' : inputValue}` : option.label,
        ),
        inputElement,
    ]);
});

const resetSaveAsName = () => {
    saveAsName.value = defaultSaveAsName.value;
};

const save = () => {
    setPreset(saveAsName.value, props.client, formClientOptions.value, saveAsName.value !== defaultSaveAsName.value);
    props.setIsOpen(false);
};

const deletePresetHandler = () => {
    deletePreset(saveAsName.value);
    setActivePresetName('chatgpt');
    props.setIsOpen(false);
};

// 监测 isOpen 支持
watch(() => props.isOpen, (isOpen) => {
    if (isOpen) {
        resetSaveAsName();
    }
});
// 监测 client 支持
watch(() => props.client, (client) => {
    if (client) {
        resetSaveAsName();
        const preset = getPreset(defaultSaveAsName.value)?.options;
        if (preset) {
            formClientOptions.value = JSON.parse(JSON.stringify(preset));
        } else {
            formClientOptions.value = {};
        }
    }
});
</script>

<template>
    <Dialog :open="isOpen" @close="setIsOpen(false)" class="relative z-50">
        <!-- 背景，作为面板容器的固定兄弟呈现 -->
        <div class="fixed inset-0 bg-black/30" aria-hidden="true" />

        <!-- 全屏可滚动容器 -->
        <div class="fixed inset-0 overflow-y-auto">
            <!-- 容器将面板居中 -->
            <div class="flex min-h-full items-center justify-center p-4">
                <!-- 实际对话框面板 -->
                <DialogPanel class="w-full max-w-3xl rounded text-slate-300 bg-white/10 backdrop-blur-lg p-6 shadow-lg">
                    <button
                        @click="setIsOpen(false)"
                        class="absolute top-0 right-0 m-2 text-white/60 hover:text-white/80 transition duration-300 ease-in-out"
                    >
                        <Icon name="bx:bx-x" class="text-2xl" />
                    </button>

                    <DialogTitle class="font-black text-slate-100 text-2xl flex items-center">
                        <GPTIcon
                            v-if="client === 'chatgpt'"
                            class="h-10 py-2 mr-2 block shadow transition duration-300 ease-in-out rounded-lg"
                        />
                        <GPTIcon
                            v-else-if="client === 'chatgpt-browser'"
                            class="h-10 py-2 mr-2 text-[#6ea194] block shadow transition duration-300 ease-in-out rounded-lg"
                        />
                        <BingIcon
                            v-else-if="client === 'bing'"
                            class="h-10 py-2 mr-2 block shadow transition duration-300 ease-in-out rounded-lg"
                        />
                        设置
                    </DialogTitle>

                    <DialogDescription class="mt-3">
                        <!-- 使用 generateForm 函数 -->
                        <div class="flex flex-col gap-2">
                            <template
                                v-for="(option, optionName) in availableOptions[client]"
                                :key="optionName"
                            >
                                <component :is="generateForm([option], optionName)[0]" />
                            </template>
                        </div>

                        <div class="flex flex-col sm:flex-row justify-end mt-4 gap-2">
                            <!-- 删除按钮 -->
                            <Transition name="fade">
                                <button
                                    v-if="saveAsName === defaultSaveAsName"
                                    type="button"
                                    class="
                                        flex items-center justify-center px-2 py-2 rounded bg-red-500/50 text-white/70
                                        hover:text-white/90 hover:bg-red-500/60 transition duration-300
                                    "
                                    @click="deletePresetHandler"
                                >
                                    <Icon name="bx:bx-trash" />
                                </button>
                            </Transition>
                            <!-- 另存为名称输入 -->
                            <div class="relative flex flex-col sm:flex-row items-stretch shadow-inner bg-white/5 rounded">
                                <label
                                    class="
                                        text-white/60 text-xs h-full flex items-center px-3 py-2 border-white/5
                                        border-b sm:border-r sm:border-b-0
                                    "
                                >
                                    预设名称
                                </label>
                                <input
                                    type="text"
                                    class="placeholder-white/40 bg-transparent text-slate-300 text-sm py-2 focus:outline-none pl-3 flex-1"
                                    placeholder="预设名称"
                                    v-model="saveAsName"
                                />
                                <button
                                    class="
                                        flex items-center justify-center px-3 py-2 group
                                        bg-white/5 sm:bg-transparent
                                    "
                                    @click="resetSaveAsName"
                                >
                                    <Icon name="bx:bx-reset" class="text-white/70 group-hover:text-white/90 transition" />
                                </button>
                            </div>
                            <button
                                type="button"
                                class="
                                    flex items-center justify-center gap-1 py-2
                                    text-slate-300 rounded bg-white/10
                                    transition duration-300 ease-in-out
                                    hover:bg-white/20
                                "
                                @click="save"
                            >
                                <Icon name="bx:bx-save" class="relative text-lg top-[1px] ml-4" /> <span class="mr-4">保存</span>
                            </button>
                        </div>
                        <!-- 小字 -->
                        <div class="mt-2 text-xs text-white/60 text-center sm:text-right">
                            对设置的任何更改都将不适用于现有的对话。
                        </div>
                    </DialogDescription>
                </DialogPanel>
            </div>
        </div>
    </Dialog>
</template>

<style>
select option {
    @apply text-slate-700;
}
</style>

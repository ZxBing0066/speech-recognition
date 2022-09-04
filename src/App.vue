<script setup lang="ts">
import { reactive, ref, watch } from 'vue';

const micOn = ref(false);
const SpeechRecognition = (window as any).SpeechRecognition || (window as any).webkitSpeechRecognition;
const recognition = new SpeechRecognition();

const options = ref({
    continuous: false,
    interimResults: false,
    maxAlternatives: 1,
    lang: 'zh-CN'
});

const recognitionResults = reactive<string[]>([]);

const commands = {
    刷新: () => location.reload(),
    后退: () => history.back(),
    首页: () => history.pushState(null, '/')
};

const setRecognitionOptions = (recognitionOptions: typeof options.value) => {
    recognition.lang = recognitionOptions.lang;
    recognition.continuous = recognitionOptions.continuous;
    recognition.maxAlternatives = recognitionOptions.maxAlternatives;
    recognition.interimResults = recognitionOptions.interimResults;
    console.log(recognitionOptions);
};

setRecognitionOptions(options.value);

recognition.onresult = (event: any) => {
    const l = event.results.length;
    console.log({}.toString.call(event.results));
    console.log('results:', event.results, recognition);

    const result = event.results[l - 1];
    console.log({}.toString.call(result));
    recognitionResults.push(result[0].transcript as string);

    commands[result[0].transcript.replaceAll(/[.,，。：？?!！:]/g, ' ').trim() as keyof typeof commands]?.();
};

recognition.onerror = function (event: any) {
    console.error('Error occurred in recognition: ' + event.error, event);
    recognitionResults.push(`${event.error}`);
};

const toggleMic = () => {
    if (!micOn.value) {
        recognition.start();
    } else {
        recognition.stop();
        // recognition.abort();
    }
    micOn.value = !micOn.value;
};

watch(() => ({ ...options.value }), setRecognitionOptions);
</script>

<template>
    <form>
        <div>
            <label>
                lang - 识别语言:
                <select v-bind:value="options.lang">
                    <option value="en-US">en-US</option>
                    <option value="zh-CN">zh-CN</option>
                </select>
            </label>
        </div>
        <div>
            <label>
                maxAlternatives - 供选语句数量:
                <input type="range" min="1" max="5" v-model="options.maxAlternatives" />
                <span>{{ options.maxAlternatives }}</span>
            </label>
        </div>
        <div>
            <label>
                continuous - 结果是否为连续:
                <input type="checkbox" name="continuous" v-bind:value="options.continuous" />
            </label>
        </div>
        <div>
            <label>
                interimResults - 临时结果？:
                <input type="checkbox" name="interimResults" v-bind:value="options.interimResults" />
            </label>
        </div>
    </form>
    <div>
        <img :class="{ mic: 1, active: micOn }" src="/mic-svgrepo-com.svg" alt="mic" @click="toggleMic" />
    </div>
    <p>点击麦克风图标切换语音助手开启状态，开启后说话可展示说话的内容</p>
    <p>说“刷新”让语音助手刷新页面。</p>
    <div class="output">
        <p v-for="res in recognitionResults">{{ res }}</p>
    </div>
</template>

<style scoped lang="scss">
.mic {
    width: 2rem;
    height: 2rem;
    margin: 0 1rem;
    filter: grayscale(1);
    will-change: filter;
    transition: filter 0.2s;
    cursor: pointer;
    &.active {
        filter: grayscale(0);
    }
}
form > div {
    margin-bottom: 1em;
}
.output {
    /* overflow: auto;
    max-height: 10em; */
}
</style>

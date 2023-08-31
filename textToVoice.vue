import CryptoJS from "crypto-js";
import { Base64 } from "js-base64";
import TransWorker from "./transcode.worker.js";
import { getToken } from "@/utils/auth";
import { encode } from "js-base64";
import { getLink } from "@/utils/common.js";
import Vue from "vue";
let transWorker = new TransWorker();
//语音转文本需要的函数
export const getWebSocketUrl = () => {
    // 请求地址根据语种不同变化
    var url = "wss://iat-api.xfyun.cn/v2/iat";
    var host = "iat-api.xfyun.cn";
    var apiKey = "d794d37c7f615ceefc460fa9e71b1bfd";
    var apiSecret = "YTczYTBlOTgzNGYyNzI0NjE3MDQwNTUz";
    var date = new Date().toGMTString();
    var algorithm = "hmac-sha256";
    var headers = "host date request-line";
    var signatureOrigin = `host: ${host}\ndate: ${date}\nGET /v2/iat HTTP/1.1`;
    var signatureSha = CryptoJS.HmacSHA256(signatureOrigin, apiSecret);
    var signature = CryptoJS.enc.Base64.stringify(signatureSha);
    var authorizationOrigin = `api_key="${apiKey}", algorithm="${algorithm}", headers="${headers}", signature="${signature}"`;
    var authorization = window.btoa(authorizationOrigin);
    url = `${url}?authorization=${authorization}&date=${date}&host=${host}`;
    return url;
};

//语音转文本需要的函数
export const toBase64 = (buffer) => {
    var binary = "";
    var bytes = new Uint8Array(buffer);
    var len = bytes.byteLength;
    for (var i = 0; i < len; i++) {
        binary += String.fromCharCode(bytes[i]);
    }
    return window.btoa(binary);
};

//文本转语音需要的函数
const getWebsocketUrl2 = () => {
    return new Promise((resolve, reject) => {
        var apiKey = "d794d37c7f615ceefc460fa9e71b1bfd";
        var apiSecret = "YTczYTBlOTgzNGYyNzI0NjE3MDQwNTUz";
        var url = "wss://tts-api.xfyun.cn/v2/tts";
        var host = location.host;
        var date = new Date().toGMTString();
        var algorithm = "hmac-sha256";
        var headers = "host date request-line";
        var signatureOrigin = `host: ${host}\ndate: ${date}\nGET /v2/tts HTTP/1.1`;
        var signatureSha = CryptoJS.HmacSHA256(signatureOrigin, apiSecret);
        var signature = CryptoJS.enc.Base64.stringify(signatureSha);
        var authorizationOrigin = `api_key="${apiKey}", algorithm="${algorithm}", headers="${headers}", signature="${signature}"`;
        var authorization = window.btoa(authorizationOrigin);
        url = `${url}?authorization=${authorization}&date=${date}&host=${host}`;

        // resolve(url);
        resolve(process.env.VUE_APP_SOCKET_URL);
    });
};

//文本转语音需要的函数
export class TTSRecorder {
    audioUrl;
    constructor({
        speed = 50,
        voice = 50,
        pitch = 50,
        voiceName = "xiaoyan",
        appId = "1c1257ea",
        text = "",
        tte = "UTF8",
        defaultText = "请输入您要合成的文本"
    } = {}) {
        this.speed = speed;
        this.voice = voice;
        this.pitch = pitch;
        this.voiceName = voiceName;
        this.text = text;
        this.tte = tte;
        this.defaultText = defaultText;
        this.appId = appId;
        this.audioData = [];
        this.rawAudioData = [];
        this.audioDataOffset = 0;
        this.status = "init";

        transWorker.onmessage = (e) => {
            this.audioData.push(...e.data.data);
            this.rawAudioData.push(...e.data.rawAudioData);
        };
    }
    // 修改录音听写状态
    setStatus(status) {
        this.onWillStatusChange && this.onWillStatusChange(this.status, status);
        this.status = status;
    }
    // 设置合成相关参数
    setParams({ speed, voice, pitch, text, voiceName, tte }) {
        speed !== undefined && (this.speed = speed);
        voice !== undefined && (this.voice = voice);
        pitch !== undefined && (this.pitch = pitch);
        text && (this.text = text);
        tte && (this.tte = tte);
        voiceName && (this.voiceName = voiceName);
        this.resetAudio();
    }
    // 连接websocket
    connectWebSocket() {
        this.setStatus("ttsing");
        return getWebsocketUrl2().then((url) => {
            let ttsWS;
            if ("WebSocket" in window) {
                ttsWS = new WebSocket(url);
            } else if ("MozWebSocket" in window) {
                ttsWS = new MozWebSocket(url);
            } else {
                alert("浏览器不支持WebSocket");
                return;
            }
            this.ttsWS = ttsWS;
            ttsWS.onopen = (e) => {
                console.log("讯飞文本转语音websocket已连接成功");
                this.webSocketSend();
                this.playTimeout = setTimeout(() => {
                    this.audioPlay();
                }, 1000);
            };
            ttsWS.onmessage = (e) => {
                this.result(e.data);
            };
            ttsWS.onerror = (e) => {
                clearTimeout(this.playTimeout);
                this.setStatus("errorTTS");
                alert("WebSocket报错，请f12查看详情");
                console.error(`详情查看：${encodeURI(url.replace("wss:", "https:"))}`);
            };
            ttsWS.onclose = (e) => {
                console.log("讯飞文本转语音websocket已断开连接");
            };
        });
    }
    // 处理音频数据
    transToAudioData(audioData) {}
    // websocket发送数据
    webSocketSend() {
        var params = {
            // common: {
            //     app_id: this.appId // APPID
            // },
            // business: {
            //     aue: "raw",
            //     auf: "audio/L16;rate=16000",
            //     vcn: this.voiceName,
            //     speed: this.speed,
            //     volume: this.voice,
            //     pitch: this.pitch,
            //     bgs: 0,
            //     tte: this.tte
            // },
            // data: {
            //     status: 2,
            //     text: this.encodeText(
            //         this.text || this.defaultText,
            //         this.tte === "unicode" ? "base64&utf16le" : ""
            //     )
            // }
            Authorization: getToken(),
            cancel: 0,
            path: "/speech_generator",
            args: encode(
                JSON.stringify({
                    text: this.text || this.defaultText,
                    voice_type: this.voiceName
                })
            )
        };
        this.ttsWS.send(JSON.stringify(params));
    }
    encodeText(text, encoding) {
        switch (encoding) {
            case "utf16le": {
                let buf = new ArrayBuffer(text.length * 4);
                let bufView = new Uint16Array(buf);
                for (let i = 0, strlen = text.length; i < strlen; i++) {
                    bufView[i] = text.charCodeAt(i);
                }
                return buf;
            }
            case "buffer2Base64": {
                let binary = "";
                let bytes = new Uint8Array(text);
                let len = bytes.byteLength;
                for (let i = 0; i < len; i++) {
                    binary += String.fromCharCode(bytes[i]);
                }
                return window.btoa(binary);
            }
            case "base64&utf16le": {
                return this.encodeText(this.encodeText(text, "utf16le"), "buffer2Base64");
            }
            default: {
                return Base64.encode(text);
            }
        }
    }
    // websocket接收数据的处理
    result(resultData) {
        let jsonData = JSON.parse(resultData);
        console.log("我是data", jsonData, transWorker);

        // 合成失败
        if (jsonData.code === 0) {
            alert(`合成失败: ${jsonData.code}:${jsonData.message}`);
            console.error(`${jsonData.code}:${jsonData.message}`);
            this.resetAudio();
            return;
        }
        console.log("====666message======  ", jsonData.data.audio);

        transWorker.postMessage(jsonData.data.audio);
        // this.audioData.push(...jsonData.data.audio);

        // if (jsonData.code === 0 && jsonData.data.status === 2) {
        //     console.log('我是data', jsonData, transWorker)
        //     this.ttsWS.close();
        // }

        if (jsonData.code == 1 && jsonData.data.finish_reason === "stop") {
            console.log(jsonData.data.audio_url);
            Vue.prototype.$bus.emit("sendFinished", getLink(jsonData.data.audio_url));
            this.ttsWS.close();
        }
    }
    // 重置音频数据
    resetAudio() {
        this.audioStop();
        this.setStatus("init");
        this.audioDataOffset = 0;
        this.audioData = [];
        this.rawAudioData = [];
        this.ttsWS && this.ttsWS.close();
        clearTimeout(this.playTimeout);
    }
    // 音频初始化
    audioInit() {
        let AudioContext = window.AudioContext || window.webkitAudioContext;
        if (AudioContext) {
            this.audioContext = new AudioContext();
            this.audioContext.resume();
            this.audioDataOffset = 0;
        }
    }
    // 音频播放
    audioPlay() {
        this.setStatus("play");
        let audioData = this.audioData.slice(this.audioDataOffset);
        this.audioDataOffset += audioData.length;
        let audioBuffer = this.audioContext.createBuffer(1, audioData.length, 22050);
        let nowBuffering = audioBuffer.getChannelData(0);
        if (audioBuffer.copyToChannel) {
            audioBuffer.copyToChannel(new Float32Array(audioData), 0, 0);
        } else {
            for (let i = 0; i < audioData.length; i++) {
                nowBuffering[i] = audioData[i];
            }
        }
        let bufferSource = (this.bufferSource = this.audioContext.createBufferSource());
        bufferSource.buffer = audioBuffer;
        bufferSource.connect(this.audioContext.destination);
        bufferSource.start();
        bufferSource.onended = (event) => {
            if (this.status !== "play") {
                return;
            }
            if (this.audioDataOffset < this.audioData.length) {
                this.audioPlay();
            } else {
                this.audioStop();
            }
        };
    }
    // 音频播放结束
    audioStop() {
        this.setStatus("endPlay");
        clearTimeout(this.playTimeout);
        this.audioDataOffset = 0;
        if (this.bufferSource) {
            try {
                this.bufferSource.stop();
            } catch (e) {
                console.log(e);
            }
        }
    }
    start() {
        if (this.audioData.length) {
            this.audioPlay();
        } else {
            if (!this.audioContext) {
                this.audioInit();
            }
            if (!this.audioContext) {
                alert("该浏览器不支持webAudioApi相关接口");
                return;
            }
            this.connectWebSocket();
        }
    }
    stop() {
        this.audioStop();
    }

    sendFinish(params) {
        if (this.ttsWS && this.ttsWS.readyState === this.ttsWS.OPEN) {
            this.ttsWS.send(JSON.stringify(params));
        }
    }
}

<template>
	<div class="page_container">
		<div class="left_box" ref="leftBox">
			<div class="left_inner_box">
				<v-md-editor
					v-model="message"
					left-toolbar=""
					right-toolbar=""
					placeholder="请输入需要内容"
					@change="listenChanged"
					mode="edit"
				/>

				<div class="left_box_top">
					<jumpPageNav />

					<!-- 从上传组件中拿到解析的结果 -->
					<upload @sendUploadResponse="message = $event" />
				</div>

				<!-- 记录多少字节以及清除文字 -->
				<div class="howmuch_and_clear_text">
					<howmuchAndClearText
						:text="message"
						:needTokenCompute="false"
						maxLen="2000字符"
						@clear="clearMessage()"
						@sendMaxLimit="message = $event"
					/>
				</div>
			</div>

			<!-- 左侧底部盒子 -->
			<div class="left_bottom">
				<el-button
					type="primary"
					v-if="!isXunfeiAudioPlaying"
					round
					@click="audioSynthesis"
					size="mini"
				>
					生成
				</el-button>

				<el-button type="primary" v-else round @click="stopAxios()" size="mini"
					>终止</el-button
				>

				<!-- 历史记录按钮 -->
				<history category="speech_generator" />

				<div class="output_languague">
					<span>声色：</span>
					<el-select v-model="audio_countenance" size="mini">
						<el-option
							v-for="({ label, value }, index) in audioCountenanceList"
							:label="label"
							:value="value"
							:key="index"
						>
						</el-option>
					</el-select>
				</div>
			</div>
		</div>
		<div class="slide_bar_box">
			<div class="slide_bar" ref="slideBar"></div>
		</div>
		<div class="right_box">
			<div class="md_box">
				<textLoading v-if="isShowLoading" />
				<div v-else class="audioPlay" @click="playAudio">
					<div style="display: none" v-if="audioUrl && audioUrl.length > 0">
						<audio controls="controls" ref="audioPlay" @ended="audioEnded">
							<source :src="audioUrl || ''" />
						</audio>
					</div>
					<div
						class="audio-detail-msg"
						v-if="needSendXunfeiArr && needSendXunfeiArr.length > 0"
					>
						<div
							class="audio-style"
							:class="{
								'add-animation': isXunfeiAudioPlaying || isPlay
							}"
						>
							<div class="small"></div>
							<div class="middle"></div>
							<div class="large"></div>
						</div>

						<div class="duration-seconds"></div>
					</div>

					<p
						v-if="needSendXunfeiArr && needSendXunfeiArr.length > 0"
						class="audio-name-style"
					>
						<span>{{ audioUrlName }}</span>
					</p>
				</div>
			</div>

			<!-- 右上角下载组件 -->
			<div class="down_load">
				<download :downloadContent="audioUrl" :fileType="fileType" />
			</div>
		</div>
	</div>
</template>

<script>
import download from "../components/download.vue";
import history from "../components/history.vue";
import jumpPageNav from "../components/jumpPageNav.vue";
import upload from "../components/upload.vue";

import dragMixin from "@/mixins/dragMixin";
import howmuchAndClearText from "../components/howmuchAndClearText.vue";
import { TTSRecorder } from "@/libs/utils";
import { getToken } from "@/utils/auth";
import { getLink } from "@/utils/common.js";

export default {
	mixins: [dragMixin("slideBar", "leftBox")],
	components: {
		history,
		upload,
		jumpPageNav,
		download,
		howmuchAndClearText
	},

	data() {
		return {
			isShowLoading: false,
			message: `中信证券股份有限公司成立于1995年10月，2003年在上海证券交易所挂牌上市交易，2011年在香港联合交易所挂牌上市交易，是中国第一家A+H股上市的证券公司，第一大股东为中国中信有限公司。公司以助力资本市场功能提升、服务经济高质量发展为使命，致力于成为全球客户最为信赖的国内领先、国际一流的中国投资银行。`,
			fileType: "audio",
			responseText: "",
			audio_countenance: "female",
			audioCountenanceList: Object.freeze([
				{
					label: "女声",
					value: "female"
				},
				{
					label: "男声",
					value: "male"
				}
			]),
			audioUrl: "",
			audioUrlName: "",
			audio_time: 0,
			xunfeiTimer: null,
			needSendXunfeiArr: [], //需要发送给讯飞的由字符串组成的数组
			isXunfeiAudioPlaying: false,
			isLoading: false,
			isPlay: false
		};
	},
	methods: {
		audioEnded() {
			this.isPlay = false;
		},

		audioSynthesis() {
			if (this.message.trim().length == 0) {
				return;
			} else if (this.isLoading) {
				return;
			} else if (this.message) {
				this.changeToAudio();
			}
		},

		clearMessage() {
			this.message = "";
			this.needSendXunfeiArr = [];
			this.isXunfeiAudioPlaying = false;
			this.pauseAudio();
		},

		listenChanged(msg) {
			if (msg.length > 2000) {
				this.message = msg.substring(0, 2000);
			}
		},

		changeToAudio() {
			this.isLoading = false;
			clearInterval(this.xunfeiTimer); //当其中一个websocket响应还在继续时，又点击了其他的音频播放，则关闭当前的循环定时器，不播放后续的音频了

			let needPlayAudioText = this.message.replace(/\[\d+\]\(.*?\)/g, "");

			if (needPlayAudioText.length < 1000) {
				this.needSendXunfeiArr.push(needPlayAudioText);
				//如果字符串长度小于1000,一次性发送给讯飞
				this.ttsRecorder.setParams({
					text: needPlayAudioText,
					voiceName: this.audio_countenance
				});
				this.ttsRecorder.start();
			} else {
				//否则将字符串拆分，再发送给讯飞
				this.setIntervalToClearNeedSendXunfeiArr();
				for (let i = 0; i < needPlayAudioText.length - 1; i += 1000) {
					this.needSendXunfeiArr.push(needPlayAudioText.slice(i, i + 1000));
				}
			}
		},

		setIntervalToClearNeedSendXunfeiArr() {
			this.needSendXunfeiArr = [];
			this.xunfeiTimer = setInterval(() => {
				this.continuePlayAudio();
				if (this.finish_reason === "stop" && this.needSendXunfeiArr.length === 0) {
					clearInterval(this.xunfeiTimer); //关闭循环定时器
				}
			}, 16);
		},

		continuePlayAudio() {
			if (this.isXunfeiAudioPlaying) return; //如果语音正在播放，则不往后执行
			if (!this.needSendXunfeiArr[0]) return; //如果为空数组，则不往后执行

			this.$bus.emit("toggleVoiceImg", "gif", this.voiceSpliceId);
			this.ttsRecorder.setParams({
				text: this.needSendXunfeiArr[0],
				voiceName: "aisjinger"
			});
			this.ttsRecorder.start();
			this.needSendXunfeiArr.shift(); //向讯飞发送需要转语音的文本后(数组中索引为0的元素)，将其弹出
		},

		stopAxios() {
			this.pauseAudio();
			this.closeTTSRecordSocket();
			this.isXunfeiAudioPlaying = false;
		},

		closeTTSRecordSocket() {
			this.ttsRecorder.sendFinish({
				cancel: 1,
				Authorization: getToken(),
				path: "/speech_generator"
			});
			clearInterval(this.xunfeiTimer);
		},

		playAudio() {
			console.log("================playAudio");
			if (this.audioUrl && this.audioUrl.length > 0) {
				this.isXunfeiAudioPlaying = false;
				let audio = eval("this.$refs.audioPlay");
				if (this.isPlay) {
					audio.pause();
					this.statAudio = false;
					this.isPlay = false;
				} else {
					audio.src = this.audioUrl;
					audio.play();
					this.isPlay = true;
				}
			} else {
				this.isPlay = false;
				if (this.isXunfeiAudioPlaying) {
					this.isXunfeiAudioPlaying = false;
					this.stopAxios();
				} else {
					this.isXunfeiAudioPlaying = true;
					this.audioSynthesis();
				}
			}
		},

		createTTSRecorderAndListenStatus() {
			this.ttsRecorder = new TTSRecorder();

			this.ttsRecorder.onWillStatusChange = (oldStatus, status) => {
				this.isXunfeiAudioPlaying = true;
				this.isShowLoading = false;
				this.isPlay = false;
				this.isLoading = false;
				console.log("++++++++++++++++++11111");
				if (oldStatus !== "init" && status === "endPlay") {
					this.isXunfeiAudioPlaying = false;
					console.log("++++++++++++++++++2222222");
				}

				if (oldStatus === "play" && status === "endPlay") {
					this.isXunfeiAudioPlaying = false;
					console.log("++++++++++++++++++333333");
					//this.$bus.emit("toggleVoiceImg", "static", this.voiceSpliceId); //每次音频自然播放完毕，都通知Interaction组件(孙子组件)，使其切换成静态图片
				}
			};
		},

		// 停止语音播放事件
		pauseAudio() {
			let audio = eval("this.$refs.audioPlay");
			audio.pause();
			this.isPlay = false;
		}
	},
	beforeDestroy() {
		this.ttsRecorder && this.ttsRecorder.resetAudio();
		this.stopAxios();
		this.$bus.off("sendAssistantHistory");
		this.$bus.off("sendFinished");
	},
	mounted() {
		this.$store.dispatch("closeWebsocket");

		this.createTTSRecorderAndListenStatus(); //创建讯飞本文转语音对象
		this.$bus.on("sendAssistantHistory", ({ answer, args }) => {
			this.needSendXunfeiArr = args.text;
			this.message = args.text;
			this.audio_countenance = args.voice_type;
			this.audioUrl = getLink(answer);
			this.audioUrlName = getLink(answer);
			this.isXunfeiAudioPlaying = false;
			this.pauseAudio();
		});

		this.$bus.on("sendFinished", (audioUrl) => {
			console.log("+++++++sendFinished++++++++++++", this.audioUrl);
			this.audioUrl = audioUrl;
			this.audioUrlName = this.audioUrl;
			//this.ttsRecorder.stop();
			//this.ttsRecorder && this.ttsRecorder.resetAudio();
		});
	}
};
</script>
<template>
	<view style="background:#F55944">
		<view class="content">
			<view class="content_1">
				<view class="content_1_3">众享亿家注册下载</view>
				<view class="content_1_1">
					<u--input placeholder="请输入手机号" border="surround" fontSize="28rpx" v-model="signParams.phone"
						color="#171717" />
				</view>
				<view class="content_1_1">
					<u--input placeholder="请输入验证码" border="surround" fontSize="28rpx" color="#171717" v-model="signParams.code">
						<template slot="suffix">
							<u-code ref="uCode" @change="codeChange" seconds="20" changeText="X秒重新获取" />
							<u-button @tap="getCode" :text="tips" type="success" color="#F76330" shape="circle" size="mini" />
						</template>
					</u--input>
				</view>
				<view class="content_1_1">
					<u--input placeholder="请输入密码" border="surround" fontSize="28rpx" color="#171717" v-model="signParams.password"
						type="password" />
				</view>
				<view class="content_1_1">
					<u--input placeholder="请再次输入密码" border="surround" fontSize="28rpx" color="#171717" v-model="isOkPassword"
						type="password" />
				</view>
				<view class="content_1_1">
					<u--input border="surround" fontSize="28rpx" color="#171717" :disabled="true" v-model="uuid" />
				</view>
				<view class="content_1_2" @click="register">注册并下载</view>
			</view>
		</view>
		<view class="mask" v-if="isMaskShow" />
	</view>
</template>

<script>
export default {
	data() {
		return {
			tips: "",
			signParams: {},
			uuid: "",
			isOkPassword: "",
			isMaskShow: false,
		}
	},
	methods: {
		codeChange(text) {
			this.tips = text;
		},
		getCode() {
			if (!this.signParams.phone) return this.$u.toast("请输入手机号");
			if (this.$refs.uCode.canGetCode) {
				// 模拟向后端请求验证码
				uni.showLoading({
					title: '正在获取验证码'
				})
				uni.request({
					url: 'http://zxyj.xzxiaocaihua.cn/api/AlibabaSms/sms',
					data: { mobile: this.signParams.phone, type: "register" },
					header: { 'Content-Type': 'application/x-www-form-urlencoded' },
					method: 'POST',
					success: res => {
						if (res.data.code != 1) return this.$u.toast(res.data.msg);
						uni.hideLoading();
						uni.$u.toast('验证码已发送');
						this.$refs.uCode.start();
					}
				});
			} else {
				uni.$u.toast('倒计时结束后再发送');
			}
		},
		register() {
			if (!this.signParams.phone) return this.$u.toast("请输入手机号");
			if (!this.signParams.code) return this.$u.toast("请输入验证码");
			if (!this.signParams.password) return this.$u.toast("请输入密码");
			if (!this.uuid) return this.$u.toast("请输入邀请码");
			if (this.signParams.password !== this.isOkPassword) return this.$u.toast("两次密码不一致");
			uni.showLoading({ title: '正在注册' })
			uni.request({
				url: 'http://zxyj.xzxiaocaihua.cn/api/login/register',
				data: {
					uuid: this.signParams.uuid,
					phone: this.signParams.phone,
					password: this.signParams.password,
					code: this.signParams.code,
				},
				header: { 'Content-Type': 'application/x-www-form-urlencoded' },
				method: 'POST',
				success: res => {
					if (res.data.code != 1) return this.$u.toast(res.data.msg);
					uni.hideLoading();
					uni.$u.toast('注册完成,正在为您下载app');
					window.location.href = "http://dm.xzxiaocaihua.cn/zxyj.apk";
				}
			});
		},
	},
	onLoad(option) {
		let ua = navigator.userAgent.toLowerCase();
		if (ua.match(/MicroMessenger/i) == 'micromessenger') this.isMaskShow = true;
		document.title = '众享亿家'
		this.uuid = `邀请码：${option.uuid}`
		this.signParams.uuid = option.uuid
	},
}
</script>

<style scoped lang="scss">
@import './index.scss';
</style>

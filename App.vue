<script>
	/**
	 * vuex管理登陆状态，具体可以参考官方登陆模板示例
	 */
	import {
		mapMutations
	} from 'vuex';
	import JSEncrypt from "@/common/SDK/jsencrypt";
	import Api from '@/common/api';
	export default {
		methods: {
			...mapMutations(['login']),
			async get_lock() {
				var params = {};
				// 获取服务器端公钥
				let data = await Api.apiCall('post', Api.index.get_lock, params);
				if (data) {
					var secret_key = data.secret_key;
					uni.setStorageSync('secret_key', secret_key);
				}
			},
			async check_version() {
				//uni-app判断客户端环境是 Android、iOS 还是小程序开发工具
				var phone_system = uni.getSystemInfoSync().platform;
				//升级检测数据  获取客户端版本号
				var version = (plus.runtime.version).split('.').join('');
				var versionCode = plus.runtime.versionCode;
				// plus.runtime.getProperty(plus.runtime.appid,(wgtinfo)=>{

				// console.log('****wgtinfo****',JSON.stringify(wgtinfo));
				// console.log('***name***',wgtinfo.name);//应用版本号
				// console.log('***package***',wgtinfo.package);
				// console.log('***version***',wgtinfo.version);//应用版本号d
				// console.log('***version***',wgtinfo.package);//应用版本号dd

				// })
				// 获取客户端信息 clientid 等
				// const clientInfo = plus.push.getClientInfo()
				// console.log('*****clientInfo*****',JSON.stringify(clientInfo));
				//版本检测:检查更新地址
				var params = {
					"package_name": 'com.dfk.muyunlang',
					"version_code": version,
					"app_id": plus.runtime.appid,
					"phone_system": phone_system,
				}
				let data = await Api.apiCall('post', Api.index.check_version, params);
				// is_update 是否提示更新 1，更新  ， 0不更新
				if (data.is_update) {
					// force_update 是否强制更新1.强制， 0.不强制
					uni.showModal({ //提醒用户更新d
						title: "更新提示",
						content: data.update_description,
						success: (res) => {
							if (res.confirm) {
								// App更新下载
								plus.runtime.openURL(data.apk_url);
							} else if (data.force_update == 1) {
								plus.runtime.quit();
							}

						}
					})
				}

			},
			mpUpdate(){
				const updateManager = uni.getUpdateManager();
				
				updateManager.onCheckForUpdate((res)=> {
				  // 请求完新版本信息的回调
				  console.log(res.hasUpdate);
				});
				
				updateManager.onUpdateReady((res)=> {
				  uni.showModal({
				    title: '更新提示',
				    content: '新版本已经准备好，是否重启应用？',
				    success(res) {
				      if (res.confirm) {
				        // 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
				        updateManager.applyUpdate();
				      }
				    }
				  });
				
				});
				
				updateManager.onUpdateFailed((res)=> {
				  // 新的版本下载失败
				});
			}
		},
		onLaunch() {
			var _this = this;
			var secret_key = uni.getStorageSync('secret_key');
			let privateKey = uni.getStorageSync('privateKey');
			let publicKey = uni.getStorageSync('publicKey');
			let userInfo = uni.getStorageSync('userInfo') || '';
			//升级检测数据  获取客户端版本号
			//#ifdef APP-PLUS
			this.check_version();
			//#endif   
			// #ifdef MP
				this.mpUpdate();
			// #endif
			if (!secret_key) {
				this.get_lock();
			}
			if (!publicKey || !privateKey) {
				//客户端生成秘钥对
				// 返回数组 array[0]公钥  array[1]私钥
				var crypt = new JSEncrypt({
					default_key_size: 1024
				});
				crypt.getKey();
				var crypt_publicKey = crypt.getPublicKey();
				var crypt_privateKey = crypt.getPrivateKey();
				// 去除-----*** RSA **** KEY----- 和空格换行
				crypt_publicKey = (crypt_publicKey.split('-----'))[2];
				// publicKey = publicKey.replace(/\n/g, "").replace(/\r/g, "").replace(/\t/g, "").replace(/\s*/g, "");
				crypt_privateKey = (crypt_privateKey.split('-----'))[2];
				uni.setStorageSync('publicKey', crypt_publicKey);
				uni.setStorageSync('privateKey', crypt_privateKey);
			}
			if (userInfo.sid) {
				//更新登陆状态
				uni.getStorage({
					key: 'userInfo',
					success: (res) => {
						_this.login(res.data);
					}
				});
			}
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style lang='scss'>
	/**
	 * 数字字体
	 */
	@font-face{
		font-family: 'Din';
		font-style: normal;
		font-weight: normal;
		src: url('/static/fonts/DIN-Bold.woff') format('woff');
	}
	.num-font{
		font-family: "Din";
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}
	
	/*
		全局公共样式和字体图标
	*/
	@font-face {
		font-family: yticon;
		font-weight: normal;
		font-style: normal;
		src: url('https://at.alicdn.com/t/font_1635142_p85koqkj5p.ttf') format('truetype');
	}

	.yticon {
		font-family: "yticon" !important;
		font-size: 16px;
		font-style: normal;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

	.icon-qiandao:before {
		content: "\e740";
	}

	.icon-jiazailoading-A:before {
		content: "\e8fc";
	}

	.icon-zuoshang:before {
		content: "\e613";
	}

	.icon-qia:before {
		content: "\e62c";
	}

	.icon-jia2:before {
		content: "\e60a";
	}

	.icon-huifu:before {
		content: "\e68b";
	}

	.icon-sousuo:before {
		content: "\e7ce";
	}

	.icon-qia1:before {
		content: "\e6da";
	}

	.icon-arrow-fine-up:before {
		content: "\e601";
	}

	.icon-hot:before {
		content: "\e60e";
	}

	.icon-sousuo3:before {
		content: "\e643";
	}

	.icon-lishijilu:before {
		content: "\e6b9";
	}

	.icon-zhengxinchaxun-zhifubaoceping-:before {
		content: "\e616";
	}

	.icon-naozhong:before {
		content: "\e64a";
	}

	.icon-xiatubiao--copy:before {
		content: "\e608";
	}

	.icon-shoucang_xuanzhongzhuangtai:before {
		content: "\e6a9";
	}

	.icon-jia1:before {
		content: "\e61c";
	}

	.icon-hongbao:before {
		content: "\e85b";
	}

	.icon-bangzhu1:before {
		content: "\e63d";
	}

	.icon-arrow-left-bottom:before {
		content: "\e602";
	}

	.icon-arrow-right-bottom:before {
		content: "\e603";
	}

	.icon-arrow-left-top:before {
		content: "\e604";
	}

	.icon-icon--:before {
		content: "\e744";
	}

	.icon-ef-zhanghao:before {
		content: "\e64e";
	}

	.icon-zuojiantou-up:before {
		content: "\e605";
	}

	.icon-xia:before {
		content: "\e62d";
	}

	.icon--jianhao:before {
		content: "\e60b";
	}

	.icon-weixinzhifu:before {
		content: "\e61a";
	}

	.icon-comment:before {
		content: "\e64f";
	}

	.icon-jifenqia:before {
		content: "\e635";
	}

	.icon-cash_payment:before {
		content: "\e63e";
	}

	.icon-weixin:before {
		content: "\e61f";
	}

	.icon-fenlei1:before {
		content: "\e620";
	}

	.icon-shangpinpingjia:before {
		content: "\e668";
	}

	.icon-erjiye-yucunkuan:before {
		content: "\e623";
	}

	.icon-guanyu:before {
		content: "\e640";
	}

	.icon-Group-:before {
		content: "\e688";
	}

	.icon-mima1:before {
		content: "\e651";
	}

	.icon-you:before {
		content: "\e606";
	}

	.icon-wallett:before {
		content: "\e6c9";
	}

	.icon-forward:before {
		content: "\e607";
	}

	.icon-huangguan:before {
		content: "\e681";
	}

	.icon-tuijian:before {
		content: "\e610";
	}

	.icon-bangzhu:before {
		content: "\e679";
	}

	.icon-share:before {
		content: "\e656";
	}

	.icon-qunzu1:before {
		content: "\e655";
	}

	.icon-yiguoqi:before {
		content: "\e997";
	}

	.icon-shezhi1:before {
		content: "\e61d";
	}

	.icon-zhipiao:before {
		content: "\e689";
	}

	.icon-fork:before {
		content: "\e61b";
	}

	.icon-kafei:before {
		content: "\e66a";
	}

	.icon-bukechakan:before {
		content: "\e634";
	}

	.icon-kechakan:before {
		content: "\e639";
	}

	.icon-iLinkapp-:before {
		content: "\e654";
	}

	.icon-saomiao:before {
		content: "\e60d";
	}

	.icon-shezhi:before {
		content: "\e60f";
	}

	.icon-shouhoutuikuan:before {
		content: "\e631";
	}

	.icon-gouwuche:before {
		content: "\e609";
	}

	.icon-dizhi:before {
		content: "\e614";
	}

	.icon-sousuo1:before {
		content: "\e641";
	}

	.icon-fenlei:before {
		content: "\e706";
	}

	.icon-xingxing:before {
		content: "\e70b";
	}

	.icon-tuandui:before {
		content: "\e633";
	}

	.icon-zuo:before {
		content: "\e63c";
	}

	.icon-yiguoqi1:before {
		content: "\e700";
	}

	.icon-shoucang2:before {
		content: "\e62e";
	}

	.icon-jifenqia-:before {
		content: "\e637";
	}

	.icon-shouhuodizhi:before {
		content: "\e712";
	}

	.icon-yishouhuo:before {
		content: "\e71a";
	}

	.icon-zhuanzhang:before {
		content: "\e62a";
	}

	.icon-sousuo2:before {
		content: "\e642";
	}

	.icon-gerenziliao:before {
		content: "\e657";
	}

	.icon-dianzan-ash:before {
		content: "\e617";
	}

	.icon-shouhoufuwu-zidongpingjia:before {
		content: "\e63a";
	}

	.icon-gouwux:before {
		content: "\e65f";
	}

	.icon-mima:before {
		content: "\e627";
	}

	.icon-yanjing1:before {
		content: "\e638";
	}

	.icon-fapiaoguanli:before {
		content: "\e62b";
	}

	.icon-jingxuan:before {
		content: "\e792";
	}

	.icon-yaoqing:before {
		content: "\e629";
	}

	.icon-rili:before {
		content: "\e745";
	}

	.icon---disanfangchengbenyingfufapiaozangubaozhangdan:before {
		content: "\e63f";
	}

	.icon-dianhua:before {
		content: "\e615";
	}

	.icon-iconfontshanchu1:before {
		content: "\e619";
	}

	.icon-iconfontweixin:before {
		content: "\e611";
	}

	.icon-wodeyouhuiquan:before {
		content: "\e65a";
	}

	.icon-alipay:before {
		content: "\e636";
	}

	.icon-shang:before {
		content: "\e624";
	}

	.icon-shouye:before {
		content: "\e626";
	}

	.icon-qunzu:before {
		content: "\e625";
	}

	.icon-shanchu4:before {
		content: "\e622";
	}

	.icon-1468422208back:before {
		content: "\e6ad";
	}

	.icon-ai47:before {
		content: "\e6a5";
	}

	.icon-xiaoxi:before {
		content: "\e618";
	}

	.icon-jiantour-copy:before {
		content: "\e600";
	}

	.icon-fenxiang2:before {
		content: "\e61e";
	}

	.icon-pingjia:before {
		content: "\e67b";
	}

	.icon-daifukuan:before {
		content: "\e68f";
	}

	.icon-yanjing:before {
		content: "\e628";
	}

	.icon-pinglun-copy:before {
		content: "\e612";
	}

	.icon-fenqiwodaifahuo:before {
		content: "\e6c2";
	}

	.icon-dianhua-copy:before {
		content: "\e621";
	}

	.icon-shoucang:before {
		content: "\e645";
	}

	.icon-ziliaoguanli:before {
		content: "\e644";
	}

	.icon-huodaofukuan:before {
		content: "\e6bf";
	}

	.icon-xuanzhong2:before {
		content: "\e62f";
	}

	.icon-gouwuche_:before {
		content: "\e630";
	}

	.icon-icon-test:before {
		content: "\e60c";
	}

	.icon-icon-test1:before {
		content: "\e632";
	}

	.icon-bianji:before {
		content: "\e646";
	}

	.icon-zhuanzhanghuikuan:before {
		content: "\e63b";
	}



	view,
	scroll-view,
	swiper,
	swiper-item,
	cover-view,
	cover-image,
	icon,
	text,
	rich-text,
	progress,
	button,
	checkbox,
	form,
	input,
	label,
	radio,
	slider,
	switch,
	textarea,
	navigator,
	audio,
	camera,
	image,
	video {
		box-sizing: border-box;
	}

	/* 骨架屏替代方案 */
	.Skeleton {
		background: #f3f3f3;
		padding: 20upx 0;
		border-radius: 8upx;
	}

	/* 图片载入替代方案 */
	.image-wrapper {
		font-size: 0;
		background: #f3f3f3;
		border-radius: 4px;

		image {
			width: 100%;
			height: 100%;
			transition: .6s;
			opacity: 0;

			&.loaded {
				opacity: 1;
			}
		}
	}

	.clamp {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		display: block;
	}

	.common-hover {
		background: #f5f5f5;
	}

	/*边框*/
	.b-b:after,
	.b-t:after {
		position: absolute;
		z-index: 3;
		left: 0;
		right: 0;
		height: 0;
		content: '';
		transform: scaleY(.5);
		border-bottom: 1px solid $border-color-base;
	}

	.b-b:after {
		bottom: 0;
	}

	.b-t:after {
		top: 0;
	}

	.align-t-b {
		display: flex;
		flex-flow: column wrap;
	}
	/* page{
		background-color: rgba(238,238,238,1);
	} */

	/* button样式改写 */
	uni-button,
	button {
		height: 80upx;
		line-height: 80upx;
		font-size: $font-lg + 2upx;
		font-weight: normal;

		&.no-border::before,
		&.no-border::after {
			border: 0;
		}
	}

	uni-button[type=default],
	button[type=default] {
		color: $font-color-dark;
	}
	button::after{
		content: unset;
	}

	/* input 样式 */
	.input-placeholder {
		color: #999999;
	}

	.placeholder {
		color: #999999;
	}
	.radius{
		border-radius: 20upx;
	}

	.ellipsis {
		width: auto;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		word-wrap: normal;
	}

	.ellipsis--l2 {
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-line-clamp: 2;
		-webkit-box-orient: vertical;
	}

	.ellipsis--l3 {
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-line-clamp: 3;
		-webkit-box-orient: vertical;
	}
</style>

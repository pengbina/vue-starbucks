<template>
	<div>
		<article class="wrapper">
			<nav class="nav">
				<nav-container v-on:showMenuOverlay="showMenuExec" v-show="!show_menu_bol">
					<span slot="navbody">
						<section class="navbody-menu body">
							<div class="accountbtn menubtn account">
								<h2>
									<span class="welcome">{{hello}}</span>
									<span class="nickname">{{NickName}}</span>
									&nbsp;☕
								</h2>
								<ul>
									<li @click="pageRedir(2, 'account')">
										<a href="javascript:;" :class="{'active-menu':curActive == 'account'}">
											账户一览
										</a>
									</li>
									<li @click="pageRedir(6, 'activity')">
										<a href="javascript:;" :class="{'active-menu':curActive == 'activity'}">
											消费记录
										</a>
									</li>
									<li @click="pageRedir(7, 'rewards')">
										<a href="javascript:;" :class="{'active-menu':curActive == 'rewards'}">
											星享好礼
										</a>
									</li>
									<li @click="pageRedir(8, 'profile')">
										<a href="javascript:;" :class="{'active-menu':curActive == 'profile'}">
											管理我的账户
										</a>
									</li>
									<li @click="pageRedir(5, 'starbucks-rewards')">
										<a href="javascript:;" :class="{'active-menu':curActive == 'starbucks-rewards'}">
											关于星享俱乐部
										</a>
									</li>
								</ul>
							</div>
						</section>
					</span>
				</nav-container>
				<nav-overlay v-on:closeMenuOverlay="closeMenuExec" v-show="show_menu_bol">
				</nav-overlay>
				<nav-mobile v-on:showMenuOverlay="showMenuExec"></nav-mobile>
			</nav>
			<article class="content account-summary-login" style="min-height:100vh;background: #F5F5F5;" v-if="lgMedia || !show_menu_bol">
				<rewards-record></rewards-record>
			</article>
		</article>
	</div>
</template>

<script>
	import '@/assets/css/bootstrap.min.css'
	import '@/assets/css/swiper.min.css'
	import '@/assets/css/init.css'
	import '@/assets/css/styles.css'
	import '@/assets/css/account.css'
	import '@/assets/css/userinfo.css'
	import NavContainer from '@/components/navContainer'
	import NavOverlay from '@/components/NavOverlay'
	import NavMobile from '@/components/navMobile'
	import RewardsRecord from '@/components/rewardsRecord'
	import TokenTools from '@/utils/tokenTools'
	import CookieTools from '@/utils/cookieTools'
	import axios from 'axios'

	export default {
		name: 'account',
		metaInfo: {
			title: '星享俱乐部',
			titleTemplate: '%s | 星巴克',
			meta: [
				{
					name: 'keywords',
					content: ''
				},
				{
					name: 'description',
					content: '星享俱乐部'
				}
			]
		},
		data(){
			return {
				show_menu_bol: false,
				lgMedia: window.matchMedia('(min-width: 1025px)').matches,
				curActive: 'rewards',
				welcome: [
					'早上好!',
					'上午好!',
					'中午好!',
					'下午好!',
					'晚上好!'
				],
				hello: '您好!'
			}
		},
		computed: {
			NickName(){
				return this.$store.state.NickName;
			}
		},
		mounted(){
			const _self = this;
			window.matchMedia('(min-width: 1025px)').addListener(()=>{
				_self.lgMedia = window.matchMedia('(min-width: 1025px)').matches;
			});
			this.checkLogin();
			this.trackingVisitor();
		},
		components: {
			NavContainer: NavContainer,
			NavOverlay: NavOverlay,
			NavMobile: NavMobile,
			RewardsRecord: RewardsRecord
		},
		methods: {
			showMenuExec(){
				this.show_menu_bol = true;
			},
			closeMenuExec(){
				this.show_menu_bol = false;
			},
			pageRedir(item, curName){
				if(curName){
					this.curActive = curName;
				}
				this.$store.commit('pageRedir', item);
			},
			checkLogin(){
				let ReqToken = TokenTools.TokenSetting('sbux_token_cl');
				
				if(ReqToken){
					axios.post("users/checkLogin",{
						ReqToken: ReqToken
					}).then((res)=>{
						let data = res.data;
						if(data.status == '0'){
							let res = data.result;
							this.$store.commit('updateUserInfo', res.NickName);
							this.getHelloWord();
						}else{
							this.$store.commit('updateUserInfo', '');
						}
						CookieTools.DelCookie('sbux_token_cl');
					})
				}
			},
			getHelloWord(){
				let time = new Date(),
					hours = time.getHours(),
					index = '';

				if(hours >= 0 && hours < 8) {
					index = 0;
				}else if(hours >= 8 && hours < 12) {
					index = 1;
				}else if(hours >= 12 && hours < 14) {
					index = 2;
				}else if(hours >= 14 && hours < 20) {
					index = 3;
				}else {
					index = 4;
				}

				this.hello = this.welcome[index];
			},
			trackingVisitor() {
				let storage = window.sessionStorage || null;
				if(storage) {
					let VisitorID = storage.getItem('VisitorID'),
						page = '我的账户-星享好礼';
					if(!VisitorID) return;
					axios.post('users/tracking',{
						visitorID: VisitorID,
						page: page
					})
				}
				
			}
		}
	}
</script>

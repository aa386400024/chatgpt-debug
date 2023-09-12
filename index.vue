<template>
	<div class="outer-box">
		<my-dialog
			:visible.sync="confirmdialogVisible"
			:dialogTitle="'信息确认'"
			:popupWidth="'90%'"
			@resetPopupData="resetPopupData('confirm')"
			@submitPopupData="submitPopupData('confirm')"
			@handleClose="handleClose('confirm')"
		>
			<div>
				<confirm-info
					:strategyInfoArr="[formOpts.strategyInfo.opts.formData]"
					:acctInfoArr="acctInfoArr"
					:entityArr="entityIdList"
					:riskarr="riskarr"
					:commissionArr="selectTradeKindArr"
					:strategy_type="formOpts.strategyInfo.opts.formData.strategy_type"
				>
				</confirm-info>
			</div>
		</my-dialog>
		<my-dialog
			:visible.sync="successdialogVisible"
			:dialogTitle="'创建成功'"
			:popupWidth="'90%'"
			:footerState="false"
			@handleClose="handleClose('success')"
		>
			<div>
				<div class="initStrate">
					<finish-create
						:strategyInfoArr="finishStrategyInfoArr"
						:acctInfoArr="finishAcctInfoArr"
						:entityInfoArr="finishEntityInfoArr"
						:strategy_type="formOpts.strategyInfo.opts.formData.strategy_type"
					></finish-create>
				</div>
				<p class="finish-create">
					策略初始化成功，您可前往
					<strong @click="routeJumpFun"> 【 策略总览 】 </strong>
					查看策略详情。
				</p>

				<div class="stepBtnBo">
					<el-button type="danger" @click="ContinueBtn()"> 重新创建策略 </el-button>
				</div>
			</div>
		</my-dialog>
		<div class="homeWrap" v-loading="loading" element-loading-text="策略正在初始化...">
			<t-module-form
				v-if="isRouterAlive"
				ref="sourceForm"
				:formOpts="formOpts"
				:footer="null"
				size="small"
				@validateError="validateError"
			>
				<template #entityInfo>
					<entity-info
						ref="entityInfoRef"
						@changeEntityIdList="changeEntityIdList"
					></entity-info>
				</template>
				<template #acctInfo>
					<acct-info
						ref="acctInfoRef"
						@acct-type-no-repeat-arr="getAcctTypeNoRepeatArr"
						@acct-arr="getAcctArr"
						:entityIdList="entityIdList"
						:strategy_type="formOpts.strategyInfo.opts.formData.strategy_type"
					></acct-info>
				</template>

				<template #ih_windcode>
					<el-autocomplete
						style="width: 220px"
						clearable
						@clear="clearSearch"
						v-model="formOpts.strategyInfo.opts.formData.ih_windname"
						:fetch-suggestions="querySearch"
						placeholder="请输入标的代码"
						:trigger-on-focus="false"
						@select="handleSearch"
						value-key="label"
					></el-autocomplete>
				</template>

				<!-- <template #ih_windname>
					<el-input
						style="width: 200px"
						v-model="formOpts.increaseHoldInfo.opts.formData.ih_windname"
						readonly="readonly"
					></el-input>
				</template> -->

				<template #ih_end_date>
					<el-date-picker
						v-model="formOpts.strategyInfo.opts.formData.ih_end_date"
						align="right"
						type="date"
						value-format="yyyy-MM-dd"
						placeholder="选择日期"
						:picker-options="pickerOptions"
					>
					</el-date-picker>
				</template>

				<template #strategy_name>
					<div style="display: flex">
						<el-autocomplete
							style="width: 140px"
							v-model="formOpts.strategyInfo.opts.formData.strategy_name"
							value-key="label"
							:disabled="formOpts.strategyInfo.opts.formData.strategy_type == 'index'"
						></el-autocomplete>
					</div>
				</template>

				<template #performance_id>
					<div style="display: flex">
						<el-autocomplete
							style="width: 180px"
							v-model="formOpts.strategyInfo.opts.formData.performance_name"
							:fetch-suggestions="querySearchQ"
							placeholder="请输入标的代码"
							:trigger-on-focus="false"
							@select="handleSearchQ"
							value-key="label"
							:disabled="formOpts.strategyInfo.opts.formData.strategy_type == 'index'"
						></el-autocomplete>
					</div>
					<!-- <div v-if="formOpts.strategyInfo.opts.formData.strategy_type != 'index'">
						
					</div>
					<div v-else>
						<div style="display: flex">
							<el-autocomplete
								style="width: 250px"
								v-model="benchmark.performance_name"
								placeholder="请输入标的代码啊"
								value-key="label"
								:disabled="
									formOpts.strategyInfo.opts.formData.strategy_type == 'index'
								"
							></el-autocomplete>
						</div>
					</div> -->
				</template>

				<template #strategy_type>
					<div style="display: flex">
						<el-select
							v-model="formOpts.strategyInfo.opts.formData.strategy_type"
							placeholder="请选择"
							style="width: 120px"
						>
							<el-option
								v-for="item in formOpts.strategyInfo.opts.listTypeInfo
									.strategyTypeList"
								:key="item.value"
								:label="item.label"
								:value="item.value"
								@click.native="changeStrategyType(item)"
							>
							</el-option>
						</el-select>
					</div>
				</template>

				<template #strategy_id>
					<div style="display: flex">
						<el-select
							style="width: 270px"
							v-model="formOpts.strategyInfo.opts.formData.strategy_id"
							placeholder="请选择"
						>
							<el-option
								v-for="item in formOpts.strategyInfo.opts.listTypeInfo
									.indexStrategyList"
								:key="item.id"
								:label="`${item.label}(${item.id})`"
								:value="item.id"
								@click.native="changeIndexStrategy(item)"
							>
							</el-option>
						</el-select>
					</div>
				</template>

				<template #tradeKind>
					<div class="trade-kind">
						<trade-kind
							ref="tradeKindRef"
							:strategy_type="formOpts.strategyInfo.opts.formData.strategy_type"
							:acctTypeNoRepeatArr="acctTypeNoRepeatArr"
							@select-trade-kind-arr="getSelectTradeKindArr"
							:disabled="formOpts.strategyInfo.opts.formData.strategy_type == 'index'"
						></trade-kind>
					</div>
				</template>
				<template #btn>
					<el-tooltip class="commission tooltip" effect="light" placement="top-end">
						<i class="el-icon-question iconStyle"></i>
						<div slot="content">
							（1）股票交易费用包括交易佣金、印花税和过户费。其中，交易佣金为全佣，包含经手费和证管费，按照成交金额的一定比例收取；印花税率为0.1%，只在卖出时收取；过户费率为0.001%，买入和卖出时均收取。<br />（2）期货交易费用包括交易佣金和手续费。其中，交易佣金按照成交金额的一定比例收取；手续费分为按成交金额收取和按成交量收取，不同品种规则不同。<br />（3）其他品种交易费用为交易佣金，交易佣金按照成交金额的一定比例收取。
						</div>
					</el-tooltip>
				</template>

				<template #commission>
					<commission
						:selectTradeKindArr="selectTradeKindArr"
						:strategy_type="formOpts.strategyInfo.opts.formData.strategy_type"
					></commission>
				</template>

				<template #riskinfo>
					<risk-info :riskarr="riskarr"></risk-info>
				</template>
			</t-module-form>
			<div class="confirmInBtn margin-top-lg">
				<el-button type="primary" size="small" @click="infoConfirmBtn()">
					信息确认
				</el-button>
			</div>
		</div>
	</div>
</template>

<script>
import StrategyMgt from "@/api/StrategyMgt";
import AcctInfo from "./components/acct_info.vue";
import TradeKind from "./components/trade_kind.vue";
import Commission from "./components/commission.vue";
import RiskInfo from "./components/riskinfo.vue";
import { judgeObject } from "@/utils";
import MyDialog from "@/components/myDialog";
import ConfirmInfo from "./components/confirm_info";
import FinishCreate from "./components/finish_create.vue";
import EntityInfo from "./components/entity_info.vue";
import constant from "@/utils/constant.js";
import SimulationMatchAPI from "@/api/SimulationMatchAPI";

export default {
	components: {
		AcctInfo,
		MyDialog,
		TradeKind,
		Commission,
		ConfirmInfo,
		RiskInfo,
		FinishCreate,
		EntityInfo
	},
	computed: {
		performanceIdList() {
			return this.$store.state.myStrategy.winCodeList;
		},

		allWindCodeList() {
			return this.$store.state.myStrategy.allWinCodeList;
		}
		/* 
		getStrategyTypeList() {
			if (this.$store.state.user.userInfoForm.department == "AI") {
				return this.formOpts.strategyInfo.opts.listTypeInfo.strategyTypeList;
			} else {
				return this.formOpts.strategyInfo.opts.listTypeInfo.strategyTypeList2;
			}
		} */
	},

	watch: {
		department: {
			handler(oldVal) {
				/* ;

						v-if="$store.state.user.userInfoForm.department == 'AI'" */
				if (oldVal == "AI") {
					this.formOpts.strategyInfo.opts.listTypeInfo.strategyTypeList = [
						{
							value: constant.investment_cons.INVESTMENT_COMMON_STRATEGY,
							label: "普通策略"
						},
						{
							value: constant.investment_cons.INVESTMENT_INCREASING_STRATEGY,
							label: "增持策略"
						},
						{
							value: constant.investment_cons.INVESTMENT_INDEX_STRATEGY,
							label: "指数策略"
						}
					];
				} else {
					this.formOpts.strategyInfo.opts.listTypeInfo.strategyTypeList = [
						{
							value: constant.investment_cons.INVESTMENT_COMMON_STRATEGY,
							label: "普通策略"
						},
						{
							value: constant.investment_cons.INVESTMENT_INDEX_STRATEGY,
							label: "指数策略"
						}
					];
				}
			},
			deep: true, // 深度监听
			immediate: true // 第一次改变就执行
		}
	},
	data() {
		return {
			loading: false,
			isRouterAlive: true,

			department: this.$store.state.user.userInfoForm
				? this.$store.state.user.userInfoForm.department
				: "",
			commonStrategyInfoList: [
				{
					label: "策略名称",
					value: "strategy_name",
					type: "input",
					comp: "el-input",
					event: "strategy_name"
				},
				{
					label: "比较基准",
					value: "performance_name",
					slotName: "performance_id"
				},
				{
					label: "策略类型",
					value: "strategy_type",
					slotName: "strategy_type"
				}
			],

			increasingStrategyInfoList: [
				{
					label: "策略名称",
					value: "strategy_name",
					type: "input",
					comp: "el-input",
					event: "strategy_name"
				},
				{
					label: "策略类型",
					value: "strategy_type",
					slotName: "strategy_type"
				},
				{
					label: "公司名称",
					value: "ih_windcode",
					slotName: "ih_windcode"
				},
				{
					label: "目标完成日期",
					value: "ih_end_date",
					slotName: "ih_end_date"
				}
			],
			indexStrategyInfoList: [
				{
					label: "策略名称",
					value: "strategy_name",
					type: "input",
					comp: "el-input",
					event: "strategy_name",
					canEdit: false,
					disabled: true,
					bind: { size: "small", clearable: false },
					slotName: "strategy_name"
				},
				{
					label: "比较基准",
					value: "performance_name",
					slotName: "performance_id"
				},
				{
					label: "策略类型",
					value: "strategy_type",
					slotName: "strategy_type"
				},
				{
					label: "引用策略",
					value: "strategy_id",
					slotName: "strategy_id"
				}
			],
			// benchmark: {
			// 	performance_id: "",
			// 	performance_name: ""
			// },
			// 表单配置
			formOpts: {
				strategyInfo: {
					key: "strategy_info",
					title: "策略信息",
					name: "strategyInfo",
					ref: null,
					widthSize: 4,

					opts: {
						labelPosition: "right",
						labelWidth: "100px",
						formData: {
							strategy_id: "",
							strategy_type: "common",
							strategy_type_name: "普通策略",
							business_type: "B",
							business_type_name: "经纪",
							strategy_name: "经纪策略",
							performance_id: "",
							performance_name: "",
							calendar: "CHN",
							calendar_name: "中国大陆",
							currency: "RMB",
							currency_name: "人民币"
						},
						fieldList: [
							{
								label: "策略名称",
								value: "strategy_name",
								type: "input",
								comp: "el-input",
								event: "strategy_name"
							},
							{
								label: "比较基准",
								value: "performance_name",
								slotName: "performance_id"
							},
							{
								label: "策略类型",
								value: "strategy_type",
								slotName: "strategy_type"
							}
						],
						rules: {
							strategy_name: [
								{ required: true, message: "请输入策略名称", trigger: "blur" },
								{ max: 10, message: "长度最大 10 个字符", trigger: "blur" }
							]
						},
						// 相关列表
						listTypeInfo: {
							performanceIdList: [
								{
									value: "COMMON",
									label: "沪深300"
								},
								{
									value: "ZZ500",
									label: "中证500"
								},
								{
									value: "ZZ800",
									label: "中证800"
								}
							],
							strategyTypeList: [],
							indexStrategyList: []
							/* strategyTypeList2: [
								{
									value: constant.investment_cons.INVESTMENT_COMMON_STRATEGY,
									label: "普通策略"
								}
							] */
						}
					}
				},

				entityInfo: {
					key: "entityInfo",
					title: "",
					name: "",
					slotName: "entityInfo"
				},
				acctInfo: {
					key: "acct_info",
					title: "账户信息",
					name: "acctInfo",
					slotName: "acctInfo"
				},
				tradeKind: {
					key: "trade_kind",
					title: "交易品种",
					name: "tradeKind",
					slotName: "tradeKind"
				},
				commission: {
					key: "commission",
					title: "交易佣金",
					name: "commission",
					slotName: "commission",
					btn: "btn"
				}
			},

			pickerOptions: {
				disabledDate(time) {
					return time.getTime() < Date.now();
				}
			},

			successdialogVisible: false,
			confirmdialogVisible: false,
			initStrategyInfoArr: [],
			acctTypeNoRepeatArr: [],
			selectTradeKindArr: [],
			riskarr: [],
			acctInfoArr: [],
			finishStrategyInfoArr: [],
			finishAcctInfoArr: [],
			finishEntityInfoArr: [],
			entityIdList: []
		};
	},
	methods: {
		changeEntityIdList(list) {
			let len = list.length;
			this.entityIdList = [];
			for (var i = 0; i < len; i++) {
				this.entityIdList.push(list[i]);
			}
		},
		// 修改标的代码输入框
		handleSearchQ(item) {
			this.formOpts.strategyInfo.opts.formData.performance_id = item.value;
			this.formOpts.strategyInfo.opts.formData.performance_name = item.label;
		},
		// 标的代码模糊查询
		querySearchQ(queryString, cb) {
			var restaurants = this.performanceIdList;
			var results = queryString
				? restaurants.filter(this.createFilterQ(queryString))
				: restaurants;
			cb(results);
		},

		clearSearch() {
			this.formOpts.strategyInfo.opts.formData.ih_windname = "";
		},
		createFilterQ(queryString) {
			return (restaurant) => {
				return restaurant.label.toLowerCase().indexOf(queryString.toLowerCase()) === 0;
			};
		},
		// 修改标的代码输入框
		handleSearch(item) {
			this.formOpts.strategyInfo.opts.formData.ih_windcode = item.value;
			this.formOpts.strategyInfo.opts.formData.ih_windname = item.name;
		},
		// 标的代码模糊查询
		querySearch(queryString, cb) {
			// var restaurants = this.allWindCodeList;
			// var results = queryString
			// 	? restaurants.filter(this.createFilter(queryString))
			// 	: restaurants;
			// cb(results);
			let params = {
				search_value: queryString
			};
			SimulationMatchAPI.getInstitutionsName(params)
				.then((res) => {
					let resp = judgeObject(res.data) ? res.data : {};
					if (resp.code == 1) {
						let data = [];
						data = judgeObject(resp.data) ? resp.data : [];
						cb(data);
					}
				})
				.catch((error) => {
					console.log(error, "error");
				});
		},
		createFilter(queryString) {
			return (restaurant) => {
				let ret =
					restaurant.name.toLowerCase().indexOf(queryString.toLowerCase()) === 0 ||
					restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0;
				return ret;
			};
		},
		changeStrategyType(item) {
			// 切换策略类型后 清空比较基准信息 重新查询， 指数策略的比较基准与增持和普通的不同
			this.formOpts.strategyInfo.opts.formData.performance_id = "";
			this.formOpts.strategyInfo.opts.formData.performance_name = "";

			if (item.value == constant.investment_cons.INVESTMENT_INCREASING_STRATEGY) {
				this.formOpts.strategyInfo.opts.formData.strategy_name = "增持策略";
				this.formOpts.strategyInfo.opts.formData.strategy_type_name = "增持策略";
				this.formOpts.entityInfo.title = "实体信息";
				this.formOpts.entityInfo.name = "entityInfo";
				this.formOpts.tradeKind.title = "";
				this.formOpts.tradeKind.name = "";
				this.formOpts.commission.title = "";
				this.formOpts.commission.name = "";
				this.formOpts.strategyInfo.opts.fieldList = this.increasingStrategyInfoList;
				this.formOpts.riskinfo = {};
				this.getSearchBenchmarkApi();
				this.$refs.acctInfoRef.adjustAcctInfoAddPosition();

				this.formOpts.strategyInfo.opts.formData.ih_windcode = "600016.SH";
				this.formOpts.strategyInfo.opts.formData.ih_windname = "中国民生银行股份有限公司";
				this.formOpts.strategyInfo.opts.formData.ih_end_date = this.initCurrentDate();
			} else if (item.value == constant.investment_cons.INVESTMENT_COMMON_STRATEGY) {
				this.formOpts.strategyInfo.opts.formData.strategy_name = "经济策略";
				this.formOpts.strategyInfo.opts.formData.strategy_type_name = "普通策略";
				this.formOpts.entityInfo.title = "";
				this.formOpts.entityInfo.name = "";
				this.formOpts.tradeKind.title = "交易品种";
				this.formOpts.tradeKind.name = "trade_kind";
				this.formOpts.commission.title = "交易佣金";
				this.formOpts.commission.name = "commission";
				this.formOpts.strategyInfo.opts.fieldList = this.commonStrategyInfoList;
				this.formOpts.riskinfo = {};
				this.getSearchBenchmarkApi();
				this.$refs.acctInfoRef.adjustAcctInfoAddPosition();

				delete this.formOpts.strategyInfo.opts.formData.ih_windcode;
				delete this.formOpts.strategyInfo.opts.formData.ih_windname;
				delete this.formOpts.strategyInfo.opts.formData.ih_end_date;
			} else {
				/* ---------------------------------- 指数策略 ---------------------------------- */
				this.getIndexStrategyListApi();
				this.formOpts.strategyInfo.opts.formData.strategy_name = "";
				this.formOpts.strategyInfo.opts.formData.strategy_type_name = "指数策略";
				this.formOpts.entityInfo.title = "";
				this.formOpts.entityInfo.name = "";
				this.formOpts.tradeKind.title = "";
				this.formOpts.tradeKind.name = "";
				this.formOpts.commission.title = "交易佣金";
				this.formOpts.commission.name = "commission";
				this.formOpts.riskinfo = {
					key: "riskinfo",
					title: "风控信息",
					name: "riskinfo",
					slotName: "riskinfo"
				};

				this.formOpts.strategyInfo.opts.fieldList = this.indexStrategyInfoList;

				delete this.formOpts.strategyInfo.opts.formData.ih_windcode;
				delete this.formOpts.strategyInfo.opts.formData.ih_windname;
				delete this.formOpts.strategyInfo.opts.formData.ih_end_date;
				this.refreshRouterPage();
			}
			//
		},

		// 修改了引用策略的策略选项
		changeIndexStrategy(item) {
			this.formOpts.strategyInfo.opts.formData.strategy_name = item.label;
			this.getIndexStrategyInfoApi();
		},

		changePerformanceId(data) {
			this.formOpts.strategyInfo.opts.formData.performance_name = data.name;
		},

		changeIncreasingWincode(data) {
			if (this.formOpts.strategyInfo) {
				this.formOpts.strategyInfo.opts.formData.ih_windname = data.codename;
			}
		},
		// 路由跳转
		routeJumpFun() {
			this.$emit("changeTab");
			this.successdialogVisible = false;
		},
		// 获取子组件去重后的账户类型数组
		getAcctTypeNoRepeatArr(arr) {
			console.log("============== 获取子组件去重后的账户类型数组 getAcctTypeNoRepeatArr");
			console.log(arr);
			this.acctTypeNoRepeatArr = arr;
			this.$nextTick(() => {
				if (this.$refs.tradeKindRef) {
					this.$refs.tradeKindRef.concatArr();
				}
			});
		},

		// 获取子组件的账户信息
		getAcctArr(arr) {
			this.acctInfoArr = arr;
		},

		getSelectTradeKindArr(arr) {
			this.selectTradeKindArr = arr;
			this.setIndexCommissionData();
		},

		setIndexCommissionData() {
			// hack 如果是 指数策略 交易佣金从 commission_info 里读取
			if (
				this.formOpts.strategyInfo.opts.formData.strategy_type ==
				constant.investment_cons.INVESTMENT_INDEX_STRATEGY
			) {
				if (this.commissionArr && this.commissionArr.length == 3) {
					// sz sh bj  bj sh sz
					this.selectTradeKindArr[0].commission = (
						this.commissionArr[2].commission * 10000
					).toFixed(1);
					this.selectTradeKindArr[1].commission = (
						this.commissionArr[1].commission * 10000
					).toFixed(1);
					this.selectTradeKindArr[2].commission = (
						this.commissionArr[0].commission * 10000
					).toFixed(1);
				}
			}
		},

		getIncreasingArr() {
			let increasingArrList = [];
			if (this.formOpts.strategyInfo) {
				increasingArrList.push(this.formOpts.strategyInfo.opts.formData);
			}
			return increasingArrList;
		},

		// 获取比较基准数据列表
		getSearchBenchmarkApi() {
			this.$store.dispatch("getWinCode").then((data) => {
				if (this.formOpts.strategyInfo.opts.formData.strategy_type == "index") return;
				this.formOpts.strategyInfo.opts.formData.performance_id =
					data[0].value || "000300.SH";
				this.formOpts.strategyInfo.opts.formData.performance_name =
					data[0].label || "000300.SH(沪深300)";
			});
		},

		// 获取比较基准数据列表
		getAllWindCodesApi() {
			this.$store.dispatch("getAllWinCode");
		},

		// 获取引用策略数据列表
		getIndexStrategyListApi() {
			let params = {
				user_id: this.$store.state.user.userId
			};
			StrategyMgt.indexStrategyListInfo(params)
				.then((res) => {
					let resp = judgeObject(res.data) ? res.data : {};
					if (resp.code == 1) {
						this.formOpts.strategyInfo.opts.listTypeInfo.indexStrategyList = resp.data;
						if (!this.formOpts.strategyInfo.opts.formData.strategy_id) {
							this.formOpts.strategyInfo.opts.formData.strategy_id = resp.data[0].id;
							this.formOpts.strategyInfo.opts.formData.strategy_name =
								resp.data[0].label;
						} else {
							resp.data.map((item) => {
								if (
									item.id == this.formOpts.strategyInfo.opts.formData.strategy_id
								) {
									this.formOpts.strategyInfo.opts.formData.strategy_id = item.id;
									this.formOpts.strategyInfo.opts.formData.strategy_name =
										item.label;
								}
							});
						}
						this.getIndexStrategyInfoApi();
					} else {
						this.$alert(resp.msg, { showConfirmButton: false });
					}
				})
				.catch((error) => {
					console.log("error: ", error);
				});
		},

		getIndexStrategyInfoApi() {
			let params = {
				user_id: this.$store.state.user.userId,
				strategy_id: this.formOpts.strategyInfo.opts.formData.strategy_id
			};
			StrategyMgt.indexStrategyInfo(params)
				.then((res) => {
					let resp = judgeObject(res.data) ? res.data : {};
					if (resp.code == 1) {
						// this.formOpts.strategyInfo.opts.formData.performance_name = "123";
						// acct_info commission_info risk_info
						// 修改账户信息数据
						this.$refs.acctInfoRef.editConfig.table.data[0].amount =
							resp.acct_info[0].amount / 10000;
						this.$refs.acctInfoRef.editConfig.table.data[0].account_type =
							resp.acct_info[0].account_type;
						this.$refs.acctInfoRef.editConfig.table.data[0].account_type_name =
							resp.acct_info[0].account_type_name;
						this.$refs.acctInfoRef.editConfig.table.data[0].acct_id =
							resp.acct_info[0].acct_id;
						this.$refs.acctInfoRef.editConfig.table.data[0].acct_mkt =
							resp.acct_info[0].acct_mkt;
						this.$refs.acctInfoRef.editConfig.table.data[0].acct_mkt_name =
							resp.acct_info[0].acct_mkt_name;
						this.$refs.acctInfoRef.editConfig.table.data[0].acct_name =
							resp.acct_info[0].acct_name;

						this.$refs.acctInfoRef.adjustAcctInfoAddPosition(resp.acct_info[0]);

						// 风险信息
						this.riskarr = resp.risk_info;
						// 交易佣金
						this.commissionArr = resp.commission_info;
						// 设置指数策略下的佣金数据信息
						this.setIndexCommissionData();

						// 比较基准信息
						// this.benchmark.performance_id = resp.benchmark[0].wincode;
						// this.benchmark.performance_name = resp.benchmark[0].code_name;
						this.formOpts.strategyInfo.opts.formData.performance_id =
							resp.benchmark[0].windcode;
						this.formOpts.strategyInfo.opts.formData.performance_name =
							resp.benchmark[0].code_name;
					} else {
						this.$alert(resp.msg, { showConfirmButton: false });
					}
				})
				.catch((error) => {
					console.log("error: ", error);
				});
		},

		createInvestmentIndexStrategy() {
			this.toTop();
			// 删除交易佣金selectFlag属性，因为是布尔值，后台无法解析
			this.selectTradeKindArr.forEach((item) => {
				if (item.selectFlag) {
					delete item.selectFlag;
				}

				if (item.commission) {
					item.commission = Number(item.commission);
				}
			});
			let strategy_info = this.formOpts.strategyInfo.opts.formData;
			strategy_info.allocation_id = "SimpleAverage";
			// strategy_info.risk_id = this.riskarr[0].risk_id ?? "";
			strategy_info.risk_id = this.riskarr[0] ? this.riskarr[0].risk_id : "";
			// 去掉比较基准字符串里的空格
			// strategy_info.performance_id = this.benchmark.performance_id;
			// strategy_info.performance_name = this.benchmark.performance_name;

			// console.log("this.$store.state.user.userId: ", this.$store.state.user.userId);
			// console.log("strategy_info: ", strategy_info);
			// console.log("this.acctInfoArr: ", this.acctInfoArr);
			// console.log("this.selectTradeKindArr: ", this.selectTradeKindArr);
			// console.log("this.riskarr: ", this.riskarr);

			let params = {
				user_id: this.$store.state.user.userId,
				strategy_info: strategy_info,
				acct_info: this.acctInfoArr,
				// this.commissionArr 查询到的佣金信息原封不动在这里
				commission: this.selectTradeKindArr,
				risk_info: this.riskarr
			};
			StrategyMgt.createInvestmentIndexStrategy(params)
				.then((res) => {
					this.loading = false;
					let resp = judgeObject(res.data) ? res.data : {};
					if (resp.code == 1) {
						this.confirmdialogVisible = false;
						this.successdialogVisible = true;
						// 创建策略后 调用自动生成目标持仓和委托的接口
						let parasOrder = {
							user_id: this.$store.state.user.userId,
							strategy_ids: [strategy_info.strategy_id]
						};
						StrategyMgt.createInvestmentIndexStrategyOrder(parasOrder)
							.then((res) => {
								console.log("res: ", res);
							})
							.catch((error) => {
								console.log("error: ", error);
							});

						this.finishStrategyInfoArr = judgeObject(resp.strategy_data)
							? resp.strategy_data
							: [];
						this.finishAcctInfoArr = judgeObject(resp.acct_data) ? resp.acct_data : [];
						this.finishIncreasingArr = judgeObject(resp.increasing_holding_data)
							? resp.increasing_holding_data
							: [];
					} else {
						this.$alert(resp.msg, { showConfirmButton: false });
					}
				})
				.catch((error) => {
					console.log("error: ", error);
				});
		},

		// 切换级联选择器时调用的方法--修复页面不刷新的bug
		refreshRouterPage() {
			this.isRouterAlive = false;
			this.$nextTick(function () {
				this.isRouterAlive = true;
			});
		},
		// 校验失败抛出事件
		validateError(e) {
			for (let n in e) {
				this.$message.error(`${this.formOpts[n].title}存在错误,请检查输入是否正确`);
			}
		},
		// 重置表单
		resetForm() {
			console.log("重置表单");
			this.$refs.sourceForm.resetFormFields();
		},
		// 清除校验
		clearValidate() {
			console.log("清除校验");
			this.$refs.sourceForm.clearValidate();
		},

		/**
		 * @description: 错误文字提示
		 * @param {*} data
		 * @return {*} text
		 */
		alertFun(data) {
			let text = "";
			// eslint-disable-next-line no-useless-escape
			const reg =
				/[`~!@#$%^&*()_\-+=<>?:"{}|,.\/;'\\[\]·~！@#￥%……&*（）——\-+={}|《》？：“”【】、；‘’，。、]/g;

			if (Array.isArray(data)) {
				for (let i = 0; i < data.length && !text; i++) {
					const item = data[i];
					text =
						item.acct_name.length > 8
							? "账户名称长度超出限制，最大8个字符。"
							: reg.test(item.acct_name)
							? "账户名称参数非法，禁止特殊字符。"
							: "";
				}
			} else if (typeof data === "string") {
				if (this.formOpts.strategyInfo.opts.formData.strategy_type == "index") {
					text = data.length > 10 ? "策略名称长度超出限制，最多10个字符。" : "";
					return text;
				}

				text =
					data.length > 10
						? "策略名称长度超出限制，最多10个字符。"
						: reg.test(data)
						? "策略名称参数非法，禁止特殊字符。"
						: "";
			}
			return text;
		},

		/**
		 * @description: 检验策略名称和账户名称
		 */
		infoConfirmBtn() {
			let strategy_id = this.formOpts.strategyInfo.opts.formData.strategy_name;
			let strategyAlertText = this.alertFun(strategy_id);
			let acctAlertText = this.alertFun(this.acctInfoArr);

			if (strategyAlertText.length > 0) {
				this.$alert(strategyAlertText, { showConfirmButton: false });
			} else if (acctAlertText.length > 0) {
				this.$alert(acctAlertText, { showConfirmButton: false });
			} else {
				this.confirmdialogVisible = true;
			}
		},

		initCurrentDate() {
			// 默认当月
			var nowDate = new Date();
			var date = {
				year: nowDate.getFullYear() + 1,
				month: nowDate.getMonth() + 1,
				day: nowDate.getDate() + 1
			};
			var dayDate =
				date.year +
				"-" +
				(date.month >= 10 ? date.month : "0" + date.month) +
				"-" +
				(date.day >= 10 ? date.day : "0" + date.day);
			return dayDate.toString();
		},

		// 点击取消的事件
		resetPopupData() {
			//  这里可重置数据
			this.confirmdialogVisible = false;
		},
		// 初始化策略按钮
		async submitPopupData(val) {
			if (val == "confirm") {
				if (this.formOpts.strategyInfo.opts.formData.strategy_type == "common") {
					this.initStrategyInfoAPI();
				} else if (this.formOpts.strategyInfo.opts.formData.strategy_type == "index") {
					this.createInvestmentIndexStrategy();
				} else {
					this.createIncreasingHoldingStrategy();
				}
			}
		},
		// 关闭弹框
		handleClose(val) {
			if (val == "confirm") {
				this.confirmdialogVisible = false;
			} else if (val == "success") {
				this.successdialogVisible = false;
			}
		},

		// 继续创建策略
		ContinueBtn() {
			this.successdialogVisible = false;
		},

		createIncreasingHoldingStrategy() {
			let strategy_info = this.formOpts.strategyInfo.opts.formData;
			// 去掉比较基准字符串里的空格
			strategy_info.performance_name = strategy_info.performance_name.replace("(", "");
			strategy_info.performance_name = strategy_info.performance_name.replace(")", "");
			let params = {
				user_id: this.$store.state.user.userId,
				strategy_info: strategy_info,
				increasing_holding_info: this.formOpts.strategyInfo.opts.formData,
				acct_info: this.acctInfoArr,
				entity_info: this.entityIdList
			};
			StrategyMgt.createStrategyByUser(params)
				.then((res) => {
					this.loading = false;
					let resp = judgeObject(res.data) ? res.data : {};
					if (resp.code == 1) {
						this.confirmdialogVisible = false;
						this.successdialogVisible = true;
						this.finishStrategyInfoArr = judgeObject(resp.strategy_data)
							? resp.strategy_data
							: [];
						this.finishAcctInfoArr = judgeObject(resp.acct_data) ? resp.acct_data : [];

						if (resp.strategy_data.length > 0) {
							this.finishEntityInfoArr = judgeObject(
								this.finishStrategyInfoArr[0].entity_info
							)
								? this.finishStrategyInfoArr[0].entity_info
								: [];
							if (
								judgeObject(this.finishStrategyInfoArr[0].increasing_holding_data)
							) {
								let holdingData =
									this.finishStrategyInfoArr[0].increasing_holding_data[0];
								this.finishStrategyInfoArr[0].ih_windcode = holdingData.ih_windcode;
								this.finishStrategyInfoArr[0].ih_windname = holdingData.ih_windname;
								this.finishStrategyInfoArr[0].ih_end_date = holdingData.ih_end_date;
							}
							console.log("finish  finish finish  ", this.finishStrategyInfoArr);
						}

						if (this.finishAcctInfoArr.length > 0) {
							this.finishAcctInfoArr.forEach((item) => {
								if (!judgeObject(item.margin_rate)) {
									item.margin_rate = "--";
								}
							});
						}
					} else if (resp.code == 2) {
						this.$alert(resp.msg, { showConfirmButton: false });
					} else {
						this.$alert(resp.msg, { showConfirmButton: false });
					}
				})
				.catch((error) => {
					console.log(error);
				});
		},

		/**
		 * @description: 初始化策略API
		 * @param {*} user_id
		 * @param {*} strategy_info
		 * @param {*} acct_info
		 * @param {*} commission
		 * @return {*}
		 */
		initStrategyInfoAPI() {
			console.log("…………………………initStrategyInfoAPI………………………………………………", this.acctInfoArr);
			this.loading = true;
			this.toTop();
			// 删除交易佣金selectFlag属性，因为是布尔值，后台无法解析
			this.selectTradeKindArr.forEach((item) => {
				if (item.selectFlag) {
					delete item.selectFlag;
				}

				if (item.commission) {
					item.commission = Number(item.commission);
				}
			});
			let strategy_info = this.formOpts.strategyInfo.opts.formData;
			// 去掉比较基准字符串里的空格
			strategy_info.performance_name = strategy_info.performance_name.replace("(", "");
			strategy_info.performance_name = strategy_info.performance_name.replace(")", "");

			let params = {
				user_id: this.$store.state.user.userId,
				strategy_info: strategy_info,
				acct_info: this.acctInfoArr,
				commission: this.selectTradeKindArr
			};

			StrategyMgt.createStrategyByUser(params)
				.then((res) => {
					this.loading = false;
					let resp = judgeObject(res.data) ? res.data : {};
					if (resp.code == 1) {
						this.confirmdialogVisible = false;
						this.successdialogVisible = true;
						this.finishStrategyInfoArr = judgeObject(resp.strategy_data)
							? resp.strategy_data
							: [];
						this.finishAcctInfoArr = judgeObject(resp.acct_data) ? resp.acct_data : [];
						this.finishIncreasingArr = judgeObject(resp.increasing_holding_data)
							? resp.increasing_holding_data
							: [];
					} else if (resp.code == 2) {
						this.$alert(resp.msg, { showConfirmButton: false });
					} else {
						this.$alert(resp.msg, { showConfirmButton: false });
					}
				})
				.catch((error) => {
					console.log(error);
				});
		},
		/**
		 * 返回顶部
		 */
		toTop() {
			document.documentElement.scrollTop = 0;
		}
	},
	mounted() {
		this.getSearchBenchmarkApi();
		this.getAllWindCodesApi();

		console.log(this.department, "department");
		if (this.$route.query && this.$route.query.strategy_id) {
			// strategy_id strategy_name user_id

			this.formOpts.strategyInfo.opts.formData.strategy_id = this.$route.query.strategy_id;
			this.formOpts.strategyInfo.opts.formData.strategy_type = "index";
			let item;
			if (this.department == "AI") {
				item = this.formOpts.strategyInfo.opts.listTypeInfo.strategyTypeList[2];
			} else {
				item = this.formOpts.strategyInfo.opts.listTypeInfo.strategyTypeList[1];
			}
			this.changeStrategyType(item);
		}
	}
};
</script>

<style lang="scss" scoped>
/deep/.el-loading-spinner {
	margin-top: -400px;
}

.el-tooltip__popper {
	max-width: 30%;
}

// 禁止编辑部分 文字颜色加深
/deep/.el-input.is-disabled .el-input__inner {
	color: #606266;
}

/deep/.el-form-item {
	flex: none !important;
	width: auto !important;
}

.commission.tooltip {
	margin-left: 2px;
}

.finish-create {
	font-size: 18px;
	text-align: center;
	margin: 20px;

	strong {
		color: $custom--color-primary;
		cursor: pointer;
	}
}

// .strategy-type {
//     margin-left: -30px;
// }
.table-temp {
	padding: 20px 10px;
}

/deep/.el-collapse-item__header {
	font-size: 14px !important;
	font-weight: bold !important;
}

/deep/ .el-input-group__append {
	padding: 0 4px;
}

/deep/.el-select .el-input__inner {
	text-align: center;
	padding: 0 25px 0 4px;
}

h2 {
	font-size: $font-size-tab-level2;
	color: $custom--color-primary;
	margin-top: 20px;
	margin-left: 10px;
	// margin-bottom: 10px;
}

.homeWrap {
	min-height: 730px;

	.t_module_form {
		overflow: hidden;

		::v-deep {
			.el-collapse {
				.el-collapse-item {
					border-radius: 4px;
					padding: 10px 2px;
					box-sizing: border-box;
					border: 1px solid #dcdfe6;
					margin-bottom: 0;
					.el-collapse-item__header {
						border-bottom: none !important;
					}
					.el-collapse-item__wrap {
						border-bottom: none;
						padding: 0 16px !important;
					}
					&.pdb0 .el-collapse-item__content {
						padding-bottom: 0 !important;
					}
					&.noTitle {
						border: none;
						padding: 0;
					}
				}
			}
		}
	}

	.trade-kind {
		display: flex;
		justify-content: center;
	}
}

.confirmInBtn {
	text-align: center;
}

.stepBtnBo {
	text-align: center;
	margin-top: 30px;

	.el-button {
		width: 200px;
	}
}

.initStrate {
	p {
		margin-top: 20px;
		display: flex;
		justify-content: center;

		strong {
			color: $custom--color-primary;
			cursor: pointer;
		}
	}
}
</style>

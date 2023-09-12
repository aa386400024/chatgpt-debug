<template>
	<div class="outer-box">
		<div class="homeWrap" v-loading="loading" element-loading-text="策略正在初始化...">
			<t-module-form
				v-if="isRouterAlive"
				ref="sourceForm"
				:formOpts="formOpts"
				:footer="null"
				size="small"
				@validateError="validateError"
			>
			</t-module-form>
		</div>
	</div>
</template>

<script>

export default {
	data() {
		return {
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
		};
	},
	methods: {
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
	},
};
</script>
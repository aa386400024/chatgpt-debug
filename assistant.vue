<!--
 * @Author: zcl ex_zhangchunlong@citics.com
 * @Date: 2023-08-22 13:17:21
 * @LastEditors: zcl ex_zhangchunlong@citics.com
 * @LastEditTime: 2023-08-29 17:02:30
 * @FilePath: /portal-fron-end/src/views/AiAssistantHome.vue
 * @Description: 

 * 
 * Copyright (c) 2023 by ${git_name_email}, All Rights Reserved. 
-->
<template>
	<div>
		<my-dialog
			:visible="warningVisible"
			:dialogTitle="dialogName"
			:popupWidth="'30%'"
			:footerState="false"
			@handleClose="handleClose()"
		>
			<div>
				<p class="warning-msg">
					您没有该板块权限，请前往
					<strong @click="handleToPersonal">【 个人中心 - 权限申请 】</strong>
					申请权限！！！
				</p>
			</div>
		</my-dialog>
		<div class="placeholder-box"></div>
		<div class="container-margin">
			<t-table
				v-if="dataLoaded"
				:key="expandedColumn"
				class="fixed-header"
				:stripe="true"
				:border="true"
				:noShowTip="true"
				:table="homeTable"
				:columns="filteredColumns"
				:cell-style="rowStyle"
				:header-cell-style="homeTable.headerStyle"
				@header-click="handleHeaderClick"
			>
			</t-table>
		</div>
	</div>
</template>

<script>
import ProductsHomeAPI from "@/api/assistant/productsHome";
import MyDialog from "@/components/myDialog";
export default {
	components: {
		MyDialog
	},
	computed: {
		// 过滤表头计算属性
		filteredColumns() {
			return this.homeTable.columns.filter(
				(col) => col.prop === "name" || this.visibleColumns.includes(col.prop)
			);
		},
		dialogName() {
			let text = "";
			if (
				this.$store.state.user.userInfoForm != null &&
				(this.$store.state.user.userInfoForm.category == 1 ||
					this.$store.state.user.userInfoForm.category == 2)
			) {
				text = "消息提示（请联系您的客户经理）";
			} else {
				text = "消息提示";
			}
			return text;
		},
		tableColumns() {
			let columns = [
				{
					prop: "name",
					label: "输入/输出",
					bind: {
						fixed: "left",
						width: "100",
						resizable: false
					},
					renderHeader: () => {
						return (
							<div class="custom-header">
								<div class="diagonal-line"></div>
								<div class="input-text">输入</div>
								<div class="output-text">输出</div>
							</div>
						);
					}
				}
			];

			return columns;
		}
	},
	data() {
		const cachedData = localStorage.getItem("ai_router_list");
		return {
			globalObject: this.$globalObject(),
			dataLoaded: false,
			warningVisible: false,
			visibleColumns: ["text", "code", "image", "video", "audio"],
			expandedColumn: null,
			homeTable: {
				border: true,
				data: cachedData ? JSON.parse(cachedData) : [],
				columns: [
					{
						prop: "name",
						label: "输入/输出",
						bind: {
							fixed: "left",
							width: "100",
							resizable: false
						},
						renderHeader: () => {
							return (
								<div class="custom-header">
									<div class="diagonal-line"></div>
									<div class="input-text">输入</div>
									<div class="output-text">输出</div>
								</div>
							);
						}
					},
					{
						label: "文本",
						prop: "text",
						bind: {
							resizable: false
						},
						render: (text, row, index) => this.renderColumn(text, row, index, "text"),
						renderHeader: this.createHeaderRenderer
					},
					{
						label: "代码",
						prop: "code",
						bind: {
							resizable: false
						},
						render: (text, row, index) => this.renderColumn(text, row, index, "code"),
						renderHeader: this.createHeaderRenderer
					},
					{
						label: "图像",
						prop: "image",
						bind: {
							resizable: false
						},
						render: (text, row, index) => this.renderColumn(text, row, index, "image"),
						renderHeader: this.createHeaderRenderer
					},
					{
						label: "语音",
						prop: "audio",
						bind: {
							resizable: false
						},
						render: (text, row, index) => this.renderColumn(text, row, index, "audio"),
						renderHeader: this.createHeaderRenderer
					},
					{
						label: "视频",
						prop: "video",
						bind: {
							resizable: false
						},
						render: (text, row, index) => this.renderColumn(text, row, index, "video"),
						renderHeader: this.createHeaderRenderer
					}
				],
				rowStyle: {
					height: "130px",
					lineHeight: "130px",
					padding: 0
					// fontSize: "16px"
				},
				headerStyle: {
					height: "60px",
					lineHeight: "60px",
					fontSize: "14px",
					background: this.$getCSSVar("--custom--color-primary"),
					color: this.$getCSSVar("--tabel-header-color")
				}
			}
		};
	},
	methods: {
		getRouterListCache() {
			if (localStorage.getItem("ai_router_list") !== "undefined") {
				return JSON.parse(localStorage.getItem("ai_router_list"));
			}
		},
		rowStyle(row) {
			if (row.column.property == "name") {
				return {
					fontSize: "14px",
					background: this.$getCSSVar("--custom--color-primary"),
					color: this.$getCSSVar("--tabel-header-color")
				};
			} else {
				return {
					// height: "300px",
					padding: 0,
					fontSize: "14px"
				};
			}
		},
		// 关闭弹窗
		handleClose() {
			this.warningVisible = false;
		},

		// 跳转到个人中心
		handleToPersonal() {
			this.$router.push({ name: "permissionApply" });
			this.warningVisible = false;
		},

		// 监听人工智能助手产品列表
		watchAssistant() {
			this.$store.watch(
				(state) => state.menu.assistant,
				(newVal) => {
					if (newVal) {
						if (localStorage.getItem("ai_router_list") != undefined) {
							this.homeTable.data = JSON.parse(
								localStorage.getItem("ai_router_list")
							);
						}
						this.getProductsApi();
					}
				}
			);
		},

		// 获取智能助手产品API
		async getProductsApi() {
			try {
				const res = await ProductsHomeAPI.getProducts({
					data_list: this.$store.state.menu.assistant
				});
				const { data } = res.data.data;
				this.$store.commit("SET_AI_ROUTER_ALL_LIST", data);
				this.homeTable.data = data || [];
			} catch (error) {
				console.error("Error fetching getProductsApi: ", error);
			}
		},

		// 创建表头渲染
		createHeaderRenderer(row) {
			const { prop, label } = row;
			const tooltipContent = this.expandedColumn === prop ? "点击收起" : "点击展开";
			return (
				<el-tooltip content={tooltipContent} placement="top" effect="light">
					<span class="">{label}</span>
				</el-tooltip>
			);
		},
		// 行渲染事件
		renderColumn(text, row, index, type) {
			const data = row[type];
			const isExpanded = this.expandedColumn === type; // 判断当前列是否被展开

			if (isExpanded && data.dropdownList) {
				return (
					<div class="row-body-expanded">
						{data.dropdownList.map((option, optionIndex) => (
							<div
								key={optionIndex}
								class="row-body"
								onClick={() =>
									this.handleSelect(
										option.value,
										option.label,
										option.description,
										index,
										type
									)
								}
							>
								<div class="text-body">
									{option.value ? (
										<div onClick={() => this.handleBlock(option)}>
											<strong>{option.label}：</strong>
											<span class="description">{option.description}</span>
										</div>
									) : null}
								</div>
							</div>
						))}
					</div>
				);
			} else {
				return (
					<div class="row-body">
						<div class="text-body">
							{data.value ? (
								<div onClick={() => this.handleBlock(data)}>
									<strong>{data.label}：</strong>
									<span class="description">{data.description}</span>
								</div>
							) : null}
						</div>
						{data.isSelect ? (
							<el-dropdown>
								<span class="el-dropdown-link">
									<i class="el-icon-arrow-right el-icon--right"></i>
								</span>
								<el-dropdown-menu slot="dropdown">
									{data.dropdownList.map((option) => (
										<el-dropdown-item
											nativeOnClick={() =>
												this.handleSelect(
													option.value,
													option.label,
													option.description,
													index,
													type
												)
											}
										>
											{option.label}
										</el-dropdown-item>
									))}
								</el-dropdown-menu>
							</el-dropdown>
						) : null}
					</div>
				);
			}
		},

		// el-dropdown点击事件
		handleSelect(value, label, description, rowIndex, type) {
			// 修改对应行和列的标题和描述
			this.homeTable.data[rowIndex][type].label = label;
			this.homeTable.data[rowIndex][type].value = value;
			this.homeTable.data[rowIndex][type].description = description;
		},

		// 点击行内某个标题文字的事件
		handleBlock(data) {
			const { plate_per, value } = data;
			if (plate_per != 0) {
				this.$router.push({
					path: `/${value}`,
					query: {
						input_and_output: `${data.input_type}-${data.output_type}`
					}
				});
			} else {
				this.warningVisible = true;
			}
			console.log(data, "handleBlock");
		},

		// 点击表头的事件
		handleHeaderClick(column) {
			if (["text", "code", "image", "video", "audio"].includes(column.property)) {
				// 如果点击的列已经是展开的列，则恢复到原始状态
				if (this.expandedColumn === column.property) {
					this.visibleColumns = ["text", "code", "image", "video", "audio"];
					this.expandedColumn = null;
				} else {
					// 否则，按照原来的逻辑展开该列
					this.visibleColumns = [column.property];
					this.expandedColumn = column.property;
				}
			} else {
				this.expandedColumn = null;
			}
		},

		/**
		 * 返回顶部
		 */
		toTop() {
			document.documentElement.scrollTop = 0;
		}
	},

	created() {
		const cachedData = localStorage.getItem("ai_router_list");
		this.dataLoaded = !!cachedData;
		this.watchAssistant();
	},

	mounted() {
		this.toTop();
		let ssoRouterQuery = this.$store.getters.ssoRouterQuery;
		if (ssoRouterQuery.code == undefined && ssoRouterQuery.state == undefined) {
			this.$store.dispatch("getUserInfo", this.$store.getters.ssoRouterQuery).then(() => {
				let obj = {
					department: this.$store.state.user.userInfoForm.department,
					currentPath: this.$route.path
				};
				this.$store.dispatch("getGroupPlateAPI", obj);
			});
		}
		this.$bus.on("handleAiAssistant", () => {
			this.visibleColumns = ["text", "code", "image", "video", "audio"];
			this.expandedColumn = null;
		});
	},

	beforeDestroy() {
		this.$bus.off("handleAiAssistant");
	}
};
</script>

<style lang="scss" scoped>
@import "@/assets/scss/mixins.scss";
$border-color: #d3d3d3;
$border-width: 1px;

::v-deep .fixed-header {
	.el-table__header-wrapper {
		position: fixed;
		background: white;
		z-index: 100;
		width: 100%;
		padding-right: 16px; // 调整滚动条宽度
	}

	// 如果有el-table的边框，也要调整边框样式
	.el-table__body-wrapper {
		border-top: none;
	}

	// 调整表头距离顶部的距离
	.el-table__header-wrapper {
		top: 60px;
	}

	// 调整表体距离顶部的距离
	.el-table__body-wrapper {
		margin-top: 60px;
	}
}

.warning-msg {
	strong {
		color: #ce2c20;
		cursor: pointer;
		font-weight: 600;
	}
}
.custom-header {
	position: relative;
	height: 50px;
	width: 100%;
	display: flex;
	align-items: center;
	justify-content: center;

	.diagonal-line {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: linear-gradient(
			to bottom left,
			transparent 49.5%,
			$border-color 49.5%,
			$border-color 50.5%,
			transparent 50.5%
		);
	}

	.input-text {
		position: absolute;
		top: 50%;
		left: 10%;
		transform: translator(-50%, -50%);
		color: --tabel-header-color;
		// background-color: white; /* To ensure the text has a background */
		padding: 0 5px; /* Some padding around the text */
	}

	.output-text {
		position: absolute;
		bottom: 50%;
		right: 10%;
		transform: translator(50%, 50%);
		color: --tabel-header-color;
		// background-color: white;
		padding: 0 5px;
	}
}

::v-deep {
	.el-table {
		border-color: $border-color;
		border-width: $border-width;

		td.el-table__cell,
		th.el-table__cell.is-leaf {
			padding: 0;
			border-bottom: $border-width solid $border-color;
		}

		th.el-table__cell > .cell {
			padding-left: 0;
			padding-right: 0;
		}

		td,
		th {
			position: relative;
			border-color: $border-color;
			border-width: $border-width;
		}
	}

	.el-table__body {
		border-color: $border-color;
	}
}

.title {
	font-size: 13px;
	font-weight: bold;
	margin-bottom: 15px;
}

.description {
	font-size: 13px;
}

.row-body {
	display: flex;
	justify-content: flex-start;
	align-items: flex-start;
	// min-height: 80px;
	padding: 10px 0;
	.text-body {
		min-height: 90px;
		max-width: 200px;
		cursor: pointer;
		text-align: left;
		@include respond-to-width("lg") {
			max-width: 300px;
		}
	}
	.el-dropdown {
		position: absolute;
		bottom: 5px;
		right: 10px;
		.el-icon-arrow-right {
			color: $custom--color-primary;
			font-weight: bold;
			font-size: 16px;
			cursor: pointer;
		}
	}
	strong {
		font-size: 13px;
		font-weight: bold;
	}
	&:hover {
		color: $custom--color-primary;
	}
}

.row-body-expanded {
	display: flex;
	justify-content: flex-start;
	cursor: pointer;
	gap: 20px;
}

::v-deep .el-table__header {
	cursor: pointer;
}
</style>
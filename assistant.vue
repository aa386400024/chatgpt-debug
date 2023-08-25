<!--
 * @Author: zcl ex_zhangchunlong@citics.com
 * @Date: 2023-08-22 13:17:21
 * @LastEditors: zcl ex_zhangchunlong@citics.com
 * @LastEditTime: 2023-08-25 13:51:39
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
                :stripe="true"
                :border="true"
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
        }
    },
    data() {
        return {
            globalObject: this.$globalObject(),
            warningVisible: false,
            visibleColumns: ["text", "code", "image", "video"],
            expandedRowIndex: null,
            expandedColumn: null,
            homeTable: {
                border: true,
                data: [],
                columns: [
                    {
                        prop: "name",
                        label: "输入/输出",
                        bind: {
                            fixed: "left",
                            width: "100"
                        },
                        renderHeader: () => {
                            return (
                                <div class="custom-header">
                                    <div class="diagonal-line"></div>
                                    <div class="input-text">输入</div>
                                    <div class="output-text">输出</div>
                                </div>
                            );
                        },
                        render: (text, row, index) => this.renderColumn(text, row, index, "name")
                    },
                    {
                        label: "文本",
                        prop: "text",
                        render: (text, row, index) => this.renderColumn(text, row, index, "text"),
                        renderHeader: this.createHeaderRenderer
                    },
                    {
                        label: "代码",
                        prop: "code",
                        render: (text, row, index) => this.renderColumn(text, row, index, "code"),
                        renderHeader: this.createHeaderRenderer
                    },
                    {
                        label: "图像",
                        prop: "image",
                        render: (text, row, index) => this.renderColumn(text, row, index, "image"),
                        renderHeader: this.createHeaderRenderer
                    },
                    {
                        label: "视频",
                        prop: "video",
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
                    height: "50px",
                    lineHeight: "50px",
                    fontSize: "14px",
                    background: this.$getCSSVar("--custom--color-primary"),
                    color: this.$getCSSVar("--tabel-header-color")
                }
            }
        };
    },
    methods: {
        rowStyle(row) {
            if (row.column.property == "name") {
                return {
                    fontSize: "14px",
                    background: this.$getCSSVar("--custom--color-primary"),
                    color: this.$getCSSVar("--tabel-header-color")
                };
            } else {
                return {
                    // height: "80px",
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
                this.homeTable.data = data.map((item) => ({ ...item, isVisible: true })) || [];
            } catch (error) {
                console.error("Error fetching getProducts: ", error);
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
            const isRowExpanded = this.expandedRowIndex === index;
            const isExpanded = this.expandedColumn === type; // 判断当前列是否被展开
            if (type === "name") {
                return (
                    <div class="row-body" onClick={() => this.expandRow(index)}>
                        <div class="text-body">
                            {data ? (
                                <div>
                                    <strong>{data}</strong>
                                </div>
                            ) : null}
                        </div>
                        {isRowExpanded ? (
                            <div class="dropdown-expanded">
                                {Object.keys(row).map((key) => {
                                    if (row[key] && row[key].dropdownList) {
                                        return row[key].dropdownList.map((option) => (
                                            <div class="dropdown-item">
                                                <strong>{option.label}：</strong>
                                                <span>{option.description}</span>
                                            </div>
                                        ));
                                    }
                                })}
                            </div>
                        ) : null}
                    </div>
                );
            } else {
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
                                                <span class="description">
                                                    {option.description}
                                                </span>
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
                                        {data.value}
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
            }
        },

        expandRow(rowIndex) {
            if (this.homeTable.data[rowIndex].isVisible) {
                this.homeTable.data.forEach((item) => {
                    this.$set(item, "isVisible", true);
                });
            } else {
                this.homeTable.data.forEach((item, index) => {
                    if (index === rowIndex) {
                        this.$set(item, "isVisible", true);
                    } else {
                        this.$set(item, "isVisible", false);
                    }
                });
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
                this.$router.push(`/${value}`);
            } else {
                this.warningVisible = true;
            }
            console.log(data, "handleBlock");
        },

        // 点击表头的事件
        handleHeaderClick(column) {
            if (column.property === "name") {
                return; // 如果点击的是输入/输出列标题，不进行任何操作
            }
            if (["text", "code", "image", "video"].includes(column.property)) {
                // 如果点击的列已经是展开的列，则恢复到原始状态
                if (this.expandedColumn === column.property) {
                    this.visibleColumns = ["text", "code", "image", "video"];
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
    }
};
</script>

<style lang="scss" scoped>
@import "@/assets/scss/mixins.scss";
$border-color: #d3d3d3;
$border-width: 1px;
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
        top: 65%;
        left: 25%;
        transform: translate(-50%, -50%);
        color: --tabel-header-color;
        // background-color: white; /* To ensure the text has a background */
        padding: 0 5px; /* Some padding around the text */
    }

    .output-text {
        position: absolute;
        bottom: 65%;
        right: 25%;
        transform: translate(50%, 50%);
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
        min-height: 65px;
        max-width: 250px;
        cursor: pointer;
        text-align: left;
        @include respond-to-width("lg") {
            max-width: 400px;
        }
    }
    .el-dropdown {
        position: absolute;
        bottom: 5px;
        right: 15px;
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
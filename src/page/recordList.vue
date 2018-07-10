<template>
    <div class="fillcontain">
        <head-top></head-top>
        <div class="table_container" style="padding-top: 10px">
            <div class="filter-container" style="padding-bottom: 45px">
                <el-button class="filter-item" type="primary" :loading="downloadLoading"
                           style="margin-left: 10px; float: right"
                           v-waves icon="el-icon-download" @click="handleDownload">导出
                </el-button>
                <el-select @change='handleFilter' style="width: 140px;float: right" class="filter-item"
                           v-model="listQuery.sort">
                    <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key">
                    </el-option>
                </el-select>
                <el-input @keyup.enter.native="handleFilter" style="width: 140px;float: right" class="filter-item"
                          :placeholder="label.search" v-model="listQuery.search">
                </el-input>
                <div class="icon-button">
                    <i class="el-icon-refresh" style="width: 40px;float: right;margin-top:4px"
                       v-waves @click="refreshData"></i>
                </div>
            </div>
            <el-table
                :data="tableData"
                style="width: 100%" align="center"
                height="700"
                highlight-current-row>   <!--固定表头-->
                <el-table-column type="expand">
                    <template scope="props">
                        <el-form label-position="left" inline class="demo-table-expand">
                            <el-form-item label="策略">
                                <span>{{ props.row.strategy_text }}</span>
                            </el-form-item>
                        </el-form>
                    </template>
                </el-table-column>

                <el-table-column v-for="item in itemtables"
                                 :label="item.label"
                                 :prop="item.key" :min-width="item.width" style="text-align: center">
                </el-table-column>

                <el-table-column label="操作" width="220px">
                    <!--scope 对 父元素遍历，scope返回的值是slot标签上返回的所有属性值，并且是一个对象的形式保存起来，获取的是一个对象-->
                    <template scope="scope">
                        <el-button
                            size="mini" v-waves
                            @click="handleEdit(scope.$index, scope.row)">编辑
                        </el-button>
                        <el-button
                            size="mini" v-waves
                            type="Success"
                            @click="handleAdd(scope.$index, scope.row)">添加
                        </el-button>
                        <el-button
                            size="mini" v-waves
                            type="danger"
                            @click="handleDelete(scope.$index, scope.row)">删除
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="Pagination">
                <el-pagination
                    background
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="listQuery.page"
                    :page-sizes="[10,20,30, 50]"
                    :page-size="listQuery.limit"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="count">
                </el-pagination>
            </div>

            <el-dialog title="修改标书信息" width="30%" :visible.sync="dialogFormVisible">
                <el-form :model="selectTable" :rules="rules" ref="selectTable">
                    <el-form-item label="标书说明" label-width="100px" prop="description">
                        <el-col :span="18">
                            <el-input v-model="selectTable.description" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="标书姓名" label-width="100px" prop="Record_name">
                        <el-col :span="18">
                            <el-input v-model="selectTable.Record_name" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="身份证号" label-width="100px" prop="ID_number">
                        <el-col :span="18">
                            <el-input v-model="selectTable.ID_number" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="标书号" label-width="100px" prop="Bid_number">
                        <el-col :span="18">
                            <el-input v-model="selectTable.Bid_number" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="标书密码" label-width="100px" prop="Bid_password">
                        <el-col :span="18">
                            <el-input  v-model="selectTable.Bid_password" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="标书状态" label-width="100px" prop="status">
                        <el-col :span="18">
                            <el-select class="filter-item" v-model="selectTable.status">
                                <el-option v-for="item in statusOptions" :key="item.key" :label="item.label"
                                           :value="item.key">
                                </el-option>
                            </el-select>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="参拍次数" label-width="100px" prop="count">
                        <el-col :span="18">
                            <el-input v-model.number="selectTable.count" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="过期时间" label-width="100px">
                        <el-col :span="18">
                            <el-form-item prop="expired_date">
                                <el-date-picker type="date" placeholder="选择日期" v-model="selectTable.expired_date"
                                                style="width: 100%;"></el-date-picker>
                            </el-form-item>
                        </el-col>
                    </el-form-item>

                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="cancelData">取 消</el-button>
                    <el-button type="primary" @click="submitForm('selectTable')">确 定</el-button>
                </div>
            </el-dialog>
        </div>
    </div>
</template>

<script>
    import headTop from '@/components/headTop';
    import {baseUrl, baseImgPath} from '@/config/env';
    import {
        getRecord,
        addRecord,
        updateRecord,
        deleteRecord,
    } from '@/api/hpData';
    import waves from '@/directive/waves'; // 水波纹指令
    export default {
        directives: {
            waves
        },
        data() {
            const checkId_number = (rule, value, callback) => {
                let reg = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
                if(reg.test(value) === false){
                    return callback(new Error('请输入正确的身份证号'));
                }
                else{
                    callback();
                }
            };
            const checkBid_number = (rule, value, callback) => {
                let reg = /(^\d{8}$)/;
                if(reg.test(value) === false){
                    return callback(new Error('请输入8位标书号'));
                }
                else{
                    callback();
                }
            };
            const checkBid_password = (rule, value, callback) => {
                let reg = /(^\d{4}$)/;
                if(reg.test(value) === false){
                    return callback(new Error('请输入4位标书密码'));
                }
                else{
                    callback();
                }
            };
            return {
                baseUrl,
                baseImgPath,
                city: {},
                offset: 0,
                count: 0,
                listQuery: {
                    page: 1,
                    limit: 20,
                    importance: undefined,
                    search: undefined,
                    type: undefined,
                    sort: 'id'
                },
                label: {
                    search: '搜索',
                },
                sortOptions: [{label: '购买日期 +', key: 'purchase_date'}, {label: '到期时间 +', key: 'expired_date'},
                    {label: '购买日期 -', key: '-purchase_date'}, {label: '到期时间 -', key: '-expired_date'},
                    {label: 'id +', key: 'id'}, {label: 'id -', key: '-id'}
                ],
            // #(('0', '未中标结束交易'), ('1', '中标完成交易'), ('2', '正常进行中'),
            //     # ('3', '失效'), ('4', '中标未完成交易'))

                tableData: [],
                currentPage: 1,
                selectTable: {},
                dialogFormVisible: false,
                categoryOptions: [],
                selectedCategory: [],
                address: {},
                downloadLoading: false,
                refreshIng: false,
                itemtables: [
                    {label: 'ID', key: 'id', width: '50px'},
                    {label: '日期', key: 'date', width: '150px'},
                    {label: '拍手姓名', key: 'hander_name', width: '90px'},
                    {label: '标书姓名', key: 'auction_name', width: '175px'},
                    {label: '标书号', key: 'Bid_number', width: '110px'},
                    {label: '结果', key: 'result', width: '100px'},
                ],



                statusOptions: [
                    {label: '未中标结束交易', key: '未中标结束交易'},
                    {label: '中标完成交易', key: '中标完成交易'},
                    {label: '正常进行中', key: '正常进行中'},
                    {label: '标书失效', key: '标书失效'},
                    {label: '中标未完成交易', key: '中标未完成交易'},
                ],
                rules: {
                    description: [
                        {required: true, message: '请输入标书描述', trigger: 'blur'},
                        {min: 2, max: 15, message: '长度在 2 到 15 个字符', trigger: 'blur'}
                    ],
                    Record_name: [
                        {required: true, message: '请输入姓名', trigger: 'blur'},
                        {min: 2, max: 4, message: '长度在 2 到 4 个字符', trigger: 'blur'}
                    ],
                    ID_number: [
                        {validator: checkId_number, trigger: 'blur'}
                    ],
                    Bid_number: [
                        {validator: checkBid_number, trigger: 'blur'}
                    ],
                    Bid_password: [
                        {validator: checkBid_password, trigger: 'blur'}
                    ],
                    count: [
                        {required: true, message: '请输入正确的次数'},
                        {type: 'number', min: 0, max: 6, message: '请输入正确的次数'}
                    ],
                    status: [
                        {required: true, message: '请选择标书状态'}
                    ],
                    expired_date: [
                        {type: 'date', required: true, message: '请选择日期', trigger: 'change'}
                    ],
                },
            };
        },
        created() {
            this.initData();
        },
        components: {
            headTop,
        },
        methods: {
            async initData() {
                try {
                    this.getRecord();

                } catch (err) {
                    console.log('获取数据失败', err);
                }
            },
            async getRecord() {
                this.refreshIng = true;
                console.log(this.listQuery);
                const res = await getRecord({
                    page: this.listQuery.page, limit: this.listQuery.limit,
                    format: 'json', search: this.listQuery.search, sort: this.listQuery.sort,
                    available: 0
                });
                if (res.status === 401) {
                    this.refreshIng = false;
                    this.$router.push('login');
                    this.$message({
                        type: 'error',
                        message: '请重新登录'
                    })
                }
                else if (res.status === 200)
                {
                    this.tableData = [];
                    this.count = res.data.count;
                    res.data.rows.forEach(item => {
                        const tableData = {};
                        tableData.id = item.id;
                        tableData.date = item.date;
                        // tableData.date_str = new Date(item.date.replace(/-/g, '/'));   // 过期时间
                        tableData.hander_name = item.hander_name;  // 描述来源
                        tableData.auction_name = item.auction_name; // 标书姓名
                        tableData.Bid_number = item.Bid_number;  // 身份证号
                        tableData.result = item.result;  // 身份证号
                        let strategy_dick = this.handlestrategy(item.strategy_dick);  // 策略
                        this.tableData.push(tableData);
                        if (strategy_dick) {
                            let strategy_type = strategy_dick['strategy_type']; //0  1   2   3
                            tableData.strategy_dick = strategy_dick;
                            console.log('fdsfsdf', strategy_type);
                            switch (strategy_type) {
                                case '0': {
                                    tableData.strategy_text = strategy_dick.strategy_description +
                                        `第一枪 ${strategy_dick['2'][1]} 加 ${strategy_dick['2'][2]} \
                                        提前${strategy_dick['2'][3]}延迟${strategy_dick['2'][4]} \
                                        强制${strategy_dick['2'][5]}`;
                                }
                                    break;
                                case '1': {
                                    tableData.strategy_text = strategy_dick.strategy_description +
                                        `第一枪 ${strategy_dick['2'][1]} 加 ${strategy_dick['2'][2]}\
                                    提前${strategy_dick['2'][3]}延迟${strategy_dick['2'][4]} \
                                    强制${strategy_dick['2'][5]} \
                                    第二枪 ${strategy_dick['2'][8]} 加 ${strategy_dick['2'][9]}\
                                    提前${strategy_dick['2'][10]}延迟${strategy_dick['2'][11]} \
                                    强制${strategy_dick['2'][12]}`;
                                }
                                    break;
                                case '2': {
                                    tableData.strategy_text = `${strategy_dick.strategy_description} \
                                    ${strategy_dick['2'][16]} 前提前${strategy_dick['2'][14]}延迟${strategy_dick['2'][15]} \
                                    ${strategy_dick['2'][19]} 前提前${strategy_dick['2'][17]}延迟${strategy_dick['2'][18]} \
                                    ${strategy_dick['2'][22]} 前提前${strategy_dick['2'][20]}延迟${strategy_dick['2'][21]} \
                                    强制${strategy_dick['2'][23]}秒`;
                                }
                                    break;
                                case '3': {
                                    tableData.strategy_text = `${strategy_dick.strategy_dick_description} \
                                    ${strategy_dick['2'][16]} 前提前${strategy_dick['2'][14]}延迟${strategy_dick['2'][15]} \
                                    ${strategy_dick['2'][19]} 前提前${strategy_dick['2'][17]}延迟${strategy_dick['2'][18]} \
                                    ${strategy_dick['2'][22]} 前提前${strategy_dick['2'][20]}延迟${strategy_dick['2'][21]} \
                                    强制${strategy_dick['2'][23]}秒`;
                                }
                                    break;
                            }
                        }
                        else {
                            tableData.strategy_text = '未定义';
                        }



                    });
                }
                else{
                    this.refreshIng = false;
                }
            },
            handleSizeChange(val) {
                this.listQuery.limit = val;
                this.listQuery.page = 1;
                this.getRecord();
            },
            handleCurrentChange(val) {
                this.listQuery.page = val;
                this.getRecord();
            },
            handleFilter() {
                this.listQuery.page = 1;
                this.getRecord();
            },
            handleCreate() {
                // this.resetTemp()
                // this.dialogStatus = 'create'
                // this.dialogFormVisible = true
                // this.$nextTick(() => {
                //     this.$refs['dataForm'].clearValidate()
                // })
            },
            handleDownload() {
                this.downloadLoading = true;
                import('@/utils/Export2Excel').then(excel => {
                    const tHeader = ['id', 'description', 'Record_name', 'ID_number', 'Bid_password', 'status', 'count', 'expired_date_str'];
                    const filterVal = ['id', 'description', 'Record_name', 'ID_number', 'Bid_password', 'status', 'count', 'expired_date_str'];
                    const data = this.formatJson(filterVal, this.tableData);
                    excel.export_json_to_excel({
                        header: tHeader,
                        data,
                        filename: 'table-list-Record'
                    });
                    this.downloadLoading = false;
                });
            },
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    console.log(valid);
                    if (valid) {
                        console.log("1234");
                        this.updateData();

                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                    console.log('ffffssss');
                });
            },
            handleEdit(index, row) {
                console.log('ff3');
                this.selectTable = Object.assign({}, row); //深拷贝
                this.dialogFormVisible = true;  //弹出对话框
            },
            handleAdd() {
                this.$router.push({path: 'addRecord'});
                // this.selectTable = Object.assign({}, row); //深拷贝
                // this.dialogFormVisible = true;  //弹出对话框
            },
            async handleDelete(index, row) {
                try {
                    const res = await deleteRecord(row.id);
                    if (res.status === 200) {
                        this.$message({
                            type: 'success',
                            message: '删除标书成功'
                        });
                        this.tableData.splice(index, 1);
                    } else {
                        this.$message({
                            type: 'error',
                            message: '操作失败'
                        });
                    }
                } catch (err) {
                    this.$message({
                        type: 'error',
                        message: '操作失败'
                    });
                    console.log('删除店铺失败');
                }
            },
            handlestrategy(stragtegy) {
                if (stragtegy == 'none') {
                    return null;
                }
                else {
                    console.log(stragtegy);
                    return JSON.parse(stragtegy);
                }
            },
            async updateData() {
                this.dialogFormVisible = false;
                try {
                    console.log(this.selectTable);
                    this.selectTable.expired_date = this.selectTable.expired_date.ymd();
                    delete this.selectTable.expired_date_str;
                    const res = await updateRecord(this.selectTable.id, this.selectTable);
                    console.log(res.status);
                    if (res.status === 200) {
                        this.$message({
                            type: 'success',
                            message: '更新标书信息成功'
                        });
                        this.getRecord();
                    } else {
                        this.$message({
                            type: 'error',
                            message: res.message
                        });
                    }
                } catch (err) {
                    console.log('更新标书信息失败', err);
                    this.$message({
                        type: error,
                        message: err
                    })
                }
            },
            cancelData() {
                this.dialogFormVisible = false;
            },
            refreshData() {
                this.getRecord();
            },
        }
    };


</script>

<style lang="less">
    @import '../style/mixin';

    .demo-table-expand {
        font-size: 0;
    }

    .demo-table-expand label {
        width: 90px;
        color: #99a9bf;
    }

    .demo-table-expand .el-form-item {
        margin-right: 0;
        margin-bottom: 0;
        width: 50%;
    }

    .table_container {
        padding: 20px;
    }

    .Pagination {
        display: flex;
        justify-content: flex-start;
        margin-top: 8px;
    }

    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }

    .avatar-uploader .el-upload:hover {
        border-color: #20a0ff;
    }

    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 120px;
        height: 120px;
        line-height: 120px;
        text-align: center;
    }

    .avatar {
        width: 120px;
        height: 120px;
        display: block;
    }

    .el-dialog--small {
        width: 30%;
    }



</style>

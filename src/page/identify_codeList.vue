<template>
    <div class="fillcontain">
        <head-top></head-top>
        <div class="table_container" style="padding-top: 10px">
            <div class="filter-container" style="padding-bottom: 45px">
                <el-button :loading="downloadLoading" icon="el-icon-download"
                           style="margin-left: 10px; float: right" type="primary"
                           @click="handleDownload">导出
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
                    <i class="el-icon-refresh" style="width: 40px;float: right;margin-top:4px" @click="refreshData"></i>
                </div>
            </div>
            <el-table
                :data="tableData"
                height="700" fit highlight-current-row>   <!--固定表头-->

                <el-table-column type="expand">
                    <template scope="props">
                        <el-form label-position="left" inline class="demo-table-expand">
                            <template v-if="props.row.auction">
                                <el-form-item label="标书id">
                                    <span>{{ props.row.auction.auction_id }}</span>
                                </el-form-item>
                                <el-form-item label="描述">
                                    <span>{{ props.row.auction.description }}</span>
                                </el-form-item>
                                <el-form-item label="标书名">
                                    <span>{{ props.row.auction.auction_name }}</span>
                                </el-form-item>
                                <el-form-item label="身份证号">
                                    <span>{{ props.row.auction.ID_number }}</span>
                                </el-form-item>
                                <el-form-item label="标书号">
                                    <span>{{ props.row.auction.Bid_number }}</span>
                                </el-form-item>
                                <el-form-item label="标书密码">
                                    <span>{{ props.row.auction.Bid_password }}</span>
                                </el-form-item>
                                <el-form-item label="参拍次数">
                                    <span>{{ props.row.auction.count }}</span>
                                </el-form-item>
                                <el-form-item label="标书到期时间">
                                    <span>{{ props.row.auction.expired_date_str }}</span>
                                </el-form-item>
                            </template>
                            <template v-else>
                                <el-form-item>
                                    <span> 未绑定任何标书 </span>
                                </el-form-item>
                            </template>
                            <el-form-item label="当前策略">
                                <span>{{ props.row.strategy_text }}</span>
                            </el-form-item>
                        </el-form>
                    </template>
                </el-table-column>
                <el-table-column
                    label="ID" width="50px"
                    prop="id">
                </el-table-column>
                <el-table-column
                    label="名称"
                    prop="bid_name">
                </el-table-column>
                <el-table-column
                    label="购买日期"
                    prop="purchase_date_str">
                </el-table-column>
                <el-table-column
                    label="到期时间"
                    prop="expired_date_str">
                </el-table-column>
                <el-table-column
                    label="激活码"
                    prop="identify_code">
                </el-table-column>
                <el-table-column
                    label="标书"
                    prop="auction_name">
                </el-table-column>
                <el-table-column label="操作" width="350px">
                    <!--scope 对 父元素遍历，scope返回的值是slot标签上返回的所有属性值，并且是一个对象的形式保存起来，获取的是一个对象-->
                    <template scope="scope">
                        <el-button
                            size="mini"
                            @click="handleEdit(scope.$index, scope.row)">编辑
                        </el-button>
                        <el-button
                            size="mini"
                            type="Success"
                            @click="handleAdd(scope.$index, scope.row)">添加
                        </el-button>
                        <el-button
                            size="mini"
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
            <el-dialog title="修改激活码信息" width="30%" :visible.sync="dialogFormVisible">
                <el-form :model="selectTable" :rules="rules" ref="selectTable">
                    <el-form-item label="名称" label-width="100px" prop="bid_name">
                        <el-col :span="18">
                            <el-input v-model="selectTable.bid_name" auto-complete="off"></el-input>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="购买日期" label-width="100px">
                        <el-col :span="18">
                            <el-form-item prop="purchase_date">
                                <el-date-picker type="date" placeholder="选择日期" v-model="selectTable.purchase_date"
                                                style="width: 100%;"></el-date-picker>
                            </el-form-item>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="过期时间" label-width="100px">
                        <el-col :span="18">
                            <el-form-item prop="expired_date">
                                <el-date-picker type="date" placeholder="选择日期"
                                                v-model="selectTable.expired_date"
                                                style="width: 100%;"></el-date-picker>
                            </el-form-item>
                        </el-col>
                    </el-form-item>
                    <el-form-item label="更换激活码" label-width="100px">
                        <el-col :span="6">
                            <el-switch v-model="change_identify_code"></el-switch>
                        </el-col>
                        <el-col class="line" :span="12">{{'当前激活码: ' + selectTable.identify_code}}</el-col>
                    </el-form-item>
                    <el-form-item label="标书" label-width="100px" placeholder="选择">
                        <el-select class="filter-item" v-model="selectTable.auction_name">
                            <el-option v-for="item in auctionOptions" :key="item.key" :label="item.label"
                                       :value="item.key">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="策略" label-width="100px">
                        <el-button @click="changeStrategy">修改策略</el-button>
                    </el-form-item>
                    <el-form-item label="当前策略" label-width="100px">
                        {{ selectTable.strategy_text }}
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="cancelData">取 消</el-button>
                    <el-button type="primary" @click="submitForm('selectTable')">确 定</el-button>
                </div>
            </el-dialog>
            <!--策略 -->
            <el-dialog title="修改策略" width="30%" :visible.sync="strategyVisable">
                <el-form :model="strategyTable" :rules="rules" ref="strategyTable">
                    <el-form-item>
                        <el-select @change="showstrategy" v-model="strategyTable.strategytype">
                            <el-option v-for="(item, key) in strategyoption" :key="key" :label="item.label"
                                       :value="item.key">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <!--第一枪出价-->
                    <template>
                        <el-form-item label="第一枪"></el-form-item>
                        <el-form-item label="出价时间">
                            <el-col :span="7">
                                <el-input-number v-model="strategyTable.chujia_time1" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0">
                                </el-input-number>
                            </el-col>
                            <el-col class="line" :span="3">秒 加价</el-col>
                            <el-col :span="8">
                                <el-input-number v-model="strategyTable.chujia_price1" :step="100"
                                                 size="s
                                                 mall" :min="300" :max="2000">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                    </template>
                    <!--第一枪提交-->
                    <template v-if="strategyTable.strategytype == '0' || strategyTable.strategytype == '1'
                                ||    strategyTable.strategytype == '3' ">
                        <el-form-item>
                            <el-col :span="7">
                                <el-select v-model="strategyTable.tijiao_diff1">
                                    <el-option size="small" v-for="(item,key) in diffoption" :label="item.label"
                                               :key="key" :value="item.key">
                                    </el-option>
                                </el-select>
                            </el-col>
                            <el-col class="line" :span="4" style="text-align: center">提交 延迟</el-col>
                            <el-col :span="6">
                                <el-input-number v-model="strategyTable.tijiao_yanchi1" :step="0.1"
                                                 size="small" :min="0.0" :max="1.5">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="强制提交">
                            <el-col :span="10">
                                <el-input-number v-model="strategyTable.tijiao_time1" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0"
                                                 :disabled="!strategyTable.forcetijiao1">
                                </el-input-number>
                            </el-col>
                            <el-col :span="6">
                                <el-switch v-model="strategyTable.forcetijiao1"></el-switch>
                            </el-col>
                        </el-form-item>
                    </template>

                    <!--智能补枪-->
                    <template v-if="strategyTable.strategytype == '0' || strategyTable.strategytype == '2' ">
                        <el-form-item label="智能补枪">
                            <el-switch v-model="strategyTable.autoprice"></el-switch>
                        </el-form-item>
                    </template>
                    <!--第二枪出价-->
                    <template v-if="strategyTable.strategytype == '1' || strategyTable.strategytype == '3' ">
                        <el-form-item label="第二枪"></el-form-item>
                        <el-form-item label="出价时间">
                            <el-col :span="7">
                                <el-input-number v-model="strategyTable.chujia_time2" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0">
                                </el-input-number>
                            </el-col>
                            <el-col class="line" :span="3">秒 加价</el-col>
                            <el-col :span="8">
                                <el-input-number v-model="strategyTable.chujia_price2" :step="100"
                                                 size="small" :min="300" :max="2000">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                    </template>
                    <!--第二枪提交-->
                    <template v-if="strategyTable.strategytype == '1' ">
                        <el-form-item>
                            <el-col :span="7">
                                <el-select v-model="strategyTable.tijiao_diff2">
                                    <el-option size="small" v-for="(item, key) in diffoption" :label="item.label"
                                               :key="key" :value="item.key">
                                    </el-option>
                                </el-select>
                            </el-col>
                            <el-col class="line" :span="4" style="text-align: center">提交 延迟</el-col>
                            <el-col :span="6">
                                <el-input-number v-model="strategyTable.tijiao_yanchi2" :step="0.1"
                                                 size="small" :min="0.0" :max="1.5">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="强制提交">
                            <el-col :span="10">
                                <el-input-number v-model="strategyTable.tijiao_time2" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0"
                                                 :disabled="!strategyTable.forcetijiao2">
                                </el-input-number>
                            </el-col>
                            <el-col :span="6">
                                <el-switch v-model="strategyTable.forcetijiao2"></el-switch>
                            </el-col>
                        </el-form-item>
                    </template>
                    <!--&lt;!&ndash;&lt;!&ndash;动态提交&ndash;&gt;&ndash;&gt;-->
                    <template v-if="strategyTable.strategytype == '2' || strategyTable.strategytype == '3' ">
                        <el-form-item label="动态提交设置"></el-form-item>
                        <el-form-item>
                            <el-col :span="7">
                                <el-input-number v-model="strategyTable.smart_time1" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0">
                                </el-input-number>
                            </el-col>
                            <el-col :span="7">
                                <el-select v-model="strategyTable.smart_diff1">
                                    <el-option size="small" v-for="item in diffoption" :label="item.label"
                                               :key="item.key" :value="item.key">
                                    </el-option>
                                </el-select>
                            </el-col>
                            <el-col class="line" :span="4" style="text-align: center">提交 延迟</el-col>
                            <el-col :span="6">
                                <el-input-number v-model="strategyTable.smart_yanchi1" :step="0.1"
                                                 size="small" :min="0.0" :max="1.5">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                        <el-form-item>
                            <el-col :span="7">
                                <el-input-number v-model="strategyTable.smart_time2" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0">
                                </el-input-number>
                            </el-col>
                            <el-col :span="7">
                                <el-select v-model="strategyTable.smart_diff2">
                                    <el-option size="small" v-for="(item, key) in diffoption" :label="item.label"
                                               :key="key" :value="item.key">
                                    </el-option>
                                </el-select>
                            </el-col>
                            <el-col class="line" :span="4" style="text-align: center">提交 延迟</el-col>
                            <el-col :span="6">
                                <el-input-number v-model="strategyTable.smart_yanchi2" :step="0.1"
                                                 size="small" :min="0.0" :max="1.5">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                        <el-form-item>
                            <el-col :span="7">
                                <el-input-number v-model="strategyTable.smart_time3" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0">
                                </el-input-number>
                            </el-col>
                            <el-col :span="7">
                                <el-select v-model="strategyTable.smart_diff3">
                                    <el-option size="small" v-for="(item, key) in diffoption" :label="item.label"
                                               :key="key" value="item.key">
                                    </el-option>
                                </el-select>
                            </el-col>
                            <el-col class="line" :span="4" style="text-align: center">提交 延迟</el-col>
                            <el-col :span="6">
                                <el-input-number v-model="strategyTable.smart_yanchi3" :step="0.1"
                                                 size="small" :min="0.0" :max="1.5">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="强制提交">
                            <el-col :span="10">
                                <el-input-number v-model="strategyTable.smart_time" :step="0.1"
                                                 size="small" :min="0.0" :max="59.0">
                                </el-input-number>
                            </el-col>
                        </el-form-item>
                        <div slot="footer" class="dialog-footer">
                            <el-button @click="cancelData">取 消</el-button>
                            <el-button type="primary" @click="closeStrategy">确 定</el-button>
                        </div>
                    </template>
                    <div slot="footer" class="dialog-footer">
                        <el-button type="primary" @click="closeStrategy">确 定</el-button>
                    </div>
                </el-form>
                <!--<div slot="footer" class="dialog-footer">-->
                <!--<el-button @click="cancelData">取 消</el-button>-->
                <!--<el-button type="primary" @click="submitForm('selectTable')">确 定</el-button>-->
                <!--</div>-->
            </el-dialog>
        </div>
    </div>
</template>

<script>
    import headTop from '@/components/headTop';
    import {baseUrl, baseImgPath} from '@/config/env';
    import {
        getIdentify_code,
        addIdentify_code,
        updateIdentify_code,
        deleteIdentify_code, getAuction,
    } from '@/api/hpData';
    import waves from '@/directive/waves'; // 水波纹指令


    export default {
        directives: {
            waves
        },
        data() {
            var checkDate1 = (rule, value, callback) => {
                console.log(value);
                if (!value) {
                    return callback(new Error('年龄不能为空'));
                }
            };
            var checkDate2 = (rule, value, callback) => {
                console.log(value);
                if (!value) {
                    return callback(new Error('年龄不能为空'));
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
                strategyoption: [
                    {label: '单枪策略', key: '0'},
                    {label: '双枪策略', key: '1'},
                    {label: '单枪动态', key: '2'},
                    {label: '双枪动态', key: '3'},
                ],
                diffoption: [
                    {label: '踩点出价', key: 0},
                    {label: '提前100', key: 100},
                    {label: '提前200', key: 200},
                    {label: '提前300', key: 300},
                ],
                type0: ['0', '1', '3'],
                type1: ['1'],
                auctionOptions: [],
                tableData: [],
                currentPage: 1,
                selectTable: {},
                strategyTable: {
                    strategytype: '0',  //初始化策略
                    //第一枪
                    chujia_time1: 48.0,
                    chujia_price1: 700,
                    tijiao_diff1: 0,
                    tijiao_yanchi1: 0.5,
                    tijiao_time1: 55.0,
                    forcetijiao1: true,
                    //补枪
                    autoprice: true,
                    //第二枪
                    chujia_time2: 50.0,
                    chujia_price2: 700,
                    tijiao_diff2: 0,
                    tijiao_yanchi2: 0.5,
                    tijiao_time2: 55.0,
                    forcetijiao2: true,
                    //动态
                    smart_diff1: 0, smart_yanchi1: 0, smart_time1: 50.0,
                    smart_diff2: 0, smart_yanchi2: 0, smart_time2: 52.0,
                    smart_diff3: 100, smart_yanchi3: 0, smart_time3: 54.0,
                    smart_time: 55.0
                },
                auction_name: '',
                dialogFormVisible: false,  //标书
                strategyVisable: false,   //策略修改
                categoryOptions: [],
                selectedCategory: [],
                address: {},
                downloadLoading: false,
                refreshIng: false,
                change_identify_code: false,

                rules: {
                    bid_name: [
                        {required: true, message: '请输入名称', trigger: 'blur'},
                        {min: 2, max: 6, message: '长度在 2 到 6 个字符', trigger: 'blur'}
                    ],
                    purchase_date: [
                        {type: 'date', required: true, message: '请选择日期', trigger: 'change'}
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
                    this.getIdentify_code();

                } catch (err) {
                    console.log('获取数据失败', err);
                }
            },

            async getIdentify_code() {
                this.refreshIng = true;
                console.log(this.listQuery);
                const identify_codes = await getIdentify_code({
                    page: this.listQuery.page, limit: this.listQuery.limit,
                    format: 'json', search: this.listQuery.search, sort: this.listQuery.sort
                });
                if (identify_codes.status === 401) {
                    this.refreshIng = false;
                    this.$router.push('login');
                    this.$message({
                        type: 'error',
                        message: '请重新登录'
                    })                }
                else if (identify_codes.status === 200)
                {
                    this.refreshIng =false;
                    this.tableData = [];
                    this.count = identify_codes.data.count;
                    identify_codes.data.rows.forEach(item => {
                        const tableData = {};
                        tableData.id = item.id;
                        tableData.bid_name = item.bid_name;
                        tableData.identify_code = item.identify_code;
                        tableData.purchase_date_str = item.purchase_date;
                        tableData.expired_date_str = item.expired_date;
                        tableData.purchase_date = new Date(item.purchase_date.replace(/-/g, '/'));
                        tableData.expired_date = new Date(item.expired_date.replace(/-/g, '/'));
                        tableData.auction_name = '无标书信息';
                        //auction
                        if (item.auction.length !== 0) {
                            let auction = item.auction[0];
                            tableData.auction = {};
                            tableData.auction_name = auction.auction_name;
                            tableData.auction.auction_id = auction.id;
                            tableData.auction.description = auction.description;
                            tableData.auction.auction_name = auction.auction_name;
                            tableData.auction.ID_number = auction.ID_number;
                            tableData.auction.Bid_number = auction.Bid_number;
                            tableData.auction.Bid_password = auction.Bid_password;
                            tableData.auction.count = auction.count;
                            tableData.auction.expired_date_str = auction.expired_date;
                            tableData.auction.expired_date = new Date(auction.expired_date.replace(/-/g, '/'));
                        }
                        //strategy
                        //{0: Array(7), 1: Array(13), 2: Array(13), 3: Array(19), 4: Array(3),
                        // yanzhengma_scale: true, strategy_description: "单枪  48秒加700截止56秒提前100",
                        // strategy_type: "0", enter_on: true}
                        let strategy_dick = this.handlestrategy(item.strategy_dick);
                        console.log(strategy_dick);
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
                        this.tableData.push(tableData);
                    });
                }
                else{
                    this.refreshIng = false;
                }
            },
            async getAuctionOption() {
                const res = await getAuction({
                    format: 'json',
                    available: 1,  //筛选未绑定的标书
                });

                if (res.status == 200) {
                    res.data.forEach(item => {
                        this.auctionOptions.push({
                            label: item.auction_name,
                            key: item.id
                        });
                    });
                }
                else {
                    console.log('初始化失败');
                }
            },
            handleSizeChange(val) {
                this.listQuery.limit = val;
                this.listQuery.page = 1;
                this.getIdentify_code();
            },
            handleCurrentChange(val) {
                this.listQuery.page = val;
                this.getIdentify_code();
            },
            handleFilter() {
                this.listQuery.page = 1;
                this.getIdentify_code();
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
                    const tHeader = ['id', 'bid_name', 'identify_code', 'purchase_date_str', 'expired_date_str', 'auction_name'];
                    const filterVal = ['id', 'bid_name', 'identify_code', 'purchase_date_str', 'expired_date_str', 'auction_name'];
                    const data = this.formatJson(filterVal, this.tableData);
                    excel.export_json_to_excel({
                        header: tHeader,
                        data,
                        filename: 'table-list'
                    });
                    this.downloadLoading = false;
                });
            },
            handleDownload() {
                this.downloadLoading = true;
                import('@/utils/Export2Excel').then(excel => {
                    const tHeader = ['id', 'bid_name', 'identify_code', 'purchase_date_str', 'expired_date_str', 'auction_name'];
                    const filterVal = ['id', 'bid_name', 'identify_code', 'purchase_date_str', 'expired_date_str', 'auction_name'];
                    const data = this.formatJson(filterVal, this.tableData);
                    excel.export_json_to_excel({
                        header: tHeader,
                        data,
                        filename: 'table-list'
                    });
                    this.downloadLoading = false;
                });
            },
            submitForm(formName) {
                this.selectTable.strategy = [];
                for (let key in this.strategyTable) {
                    this.selectTable.strategy.push(this.strategyTable[key]);
                }
                console.log(this.selectTable);
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        //判断 是否修改了绑定的标书
                        if (this.auction_name !== this.selectTable.auction_name) {
                            console.log('修改了');
                            // this.selectTable.auction_name   内存放的是标书的id
                            this.selectTable.changeauction = true;
                        }
                        else {
                            this.selectTable.changeauction = false;
                        }
                        this.updateData();
                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });

            },
            refreshData() {
                if (!this.refreshIng){
                    this.getIdentify_code();
                }
            },
            handleEdit(index, row) {
                this.selectTable = Object.assign({}, row); //深拷贝
                this.auction_name = this.selectTable.auction_name;  //存档一下
                this.dialogFormVisible = true;  //弹出对话框
                //先清空选项,然后初始化
                this.auctionOptions = [];
                this.getAuctionOption();
                //保存策略数据
                if (this.selectTable.strategy_text !== '未定义') {
                    let dick = this.selectTable.strategy_dick;
                    let temp = dick[dick.strategy_type];
                    let index = 0;
                    for (let key in this.strategyTable) {
                        this.strategyTable[key] = temp[index];
                        index++;
                    }
                }

            },
            handleAdd() {
                this.$router.push({path: 'addIdentify_code'});
                // this.selectTable = Object.assign({}, row); //深拷贝
                // this.dialogFormVisible = true;  //弹出对话框
            },
            changeStrategy() {
                this.strategyVisable = true;
                console.log(this.strategyTable);

            },
            //策略对话框确认
            closeStrategy() {
                this.strategyVisable = false;
            },
            //策略对话框取消
            cancelStrategy() {
                this.strategyVisable = false;

            },
            async handleDelete(index, row) {
                try {
                    const res = await deleteIdentify_code(row.id);
                    if (res.status === 200) {
                        this.$message({
                            type: 'success',
                            message: '删除激活码成功'
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
            async updateData() {
                this.dialogFormVisible = false;
                try {
                    console.log(this.selectTable);
                    this.selectTable.purchase_date_str = this.selectTable.purchase_date.ymd();
                    this.selectTable.expired_date_str = this.selectTable.expired_date.ymd();
                    this.selectTable.change_identify_code = this.change_identify_code;
                    let data = {'data': JSON.stringify(this.selectTable)};
                    const res = await updateIdentify_code(this.selectTable.id, data);
                    console.log(res.status);
                    if (res.status === 200) {
                        this.$message({
                            type: 'success',
                            message: '更新激活码信息成功'
                        });
                        this.getIdentify_code();
                    } else {
                        this.$message({
                            type: 'error',
                            message: res.message
                        });
                    }
                } catch (err) {
                    console.log('更新激活码信息失败', err);
                }
            },
            cancelData() {
                this.dialogFormVisible = false;
            },
            showstrategy() {
                console.log(this.strategyTable);
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
            formatJson(filterVal, jsonData) {
                return jsonData.map(v => filterVal.map(j => {
                    if (j === 'timestamp') {
                        return parseTime(v[j]);
                    } else {
                        return v[j];
                    }
                }));
            }
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

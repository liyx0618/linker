<template>
    <el-dialog class="options-center" :title="$t('server.relayCdkey')" destroy-on-close v-model="state.show" width="77rem" top="2vh">
    <div class="group-wrap">
        <div class="head">
            <div class="search flex">
                <div><span>{{$t('server.relayCdkeyUserId')}}</span> <el-input v-model="state.page.UserId" style="width:8rem" size="small" clearable @change="handleSearch" /></div>
                <div><span>{{$t('server.relayCdkeyOrderId')}}</span> <el-input v-model="state.page.OrderId" style="width:8rem" size="small" clearable @change="handleSearch" /></div>
                <div><span>{{$t('server.relayCdkeyContact')}}</span> <el-input v-model="state.page.Contact" style="width:8rem" size="small" clearable @change="handleSearch" /></div>
                <div><span>{{$t('server.relayCdkeyRemark')}}</span> <el-input v-model="state.page.Remark" style="width:8rem" size="small" clearable @change="handleSearch" /></div>
                <div>
                    <el-button size="small" @click="handleSearch()">
                        <el-icon><Search /></el-icon>
                    </el-button>
                </div>
                <div>
                    <el-button size="small" @click="state.showTest = true">
                        <el-icon><Warning /></el-icon>
                    </el-button>
                </div>
                <div>
                    <el-button size="small" type="success" @click="state.showAdd = true">
                        <el-icon><Plus /></el-icon>
                    </el-button>
                </div>
            </div>
            <Flags @change="handleFlagsChange"></Flags>
        </div>
        <el-table stripe  :data="state.list.List" border size="small" width="100%" @sort-change="handleSort">
            <el-table-column prop="Bandwidth" :label="$t('server.relayCdkeyBandwidth')" width="110" sortable="custom">
                <template #default="scope">{{ scope.row.Bandwidth }}Mbps</template>
            </el-table-column>
            <el-table-column prop="LastBytes" :label="`${$t('server.relayCdkeyBytes')}`" width="80" sortable="custom">
                <template #default="scope">
                    <p><strong>{{ parseSpeed(scope.row.LastBytes) }}</strong></p>
                    <p>{{ parseSpeed(scope.row.MaxBytes) }}</p>
                </template>
            </el-table-column>
            <el-table-column prop="PayPrice" :label="`${$t('server.relayCdkeyPay')}`" width="120" sortable="custom">
                <template #default="scope">
                    <p><strong>{{$t('server.relayCdkeyPayPrice')}}.{{ scope.row.PayPrice }}</strong>/{{$t('server.relayCdkeyPrice')}}.{{ scope.row.Price }}</p>
                    <p>{{$t('server.relayCdkeyUserPrice')}}.{{ scope.row.UserPrice }}/{{$t('server.relayCdkeyCostPrice')}}.{{ scope.row.CostPrice }}</p>
                </template>
            </el-table-column>
            <el-table-column prop="OrderId" :label="`${$t('server.relayCdkeyOrder')}`" width="180">
                <template #default="scope">
                    <p>{{ scope.row.OrderId }}</p>
                    <p>{{ scope.row.Contact }}</p>
                </template>
            </el-table-column>
            <el-table-column prop="Remark" :label="$t('server.relayCdkeyRemark')"></el-table-column>
            <el-table-column prop="EndTime" :label="`${$t('server.relayCdkeyEndTime')}`" width="140" sortable="custom">
            </el-table-column>
            <el-table-column prop="UseTime" :label="`${$t('server.relayCdkeyUseTime')}`" width="140" sortable="custom">
            </el-table-column>
            <el-table-column prop="AddTime" :label="`${$t('server.relayCdkeyAddTime')}`" width="140" sortable="custom">
            </el-table-column>
            <el-table-column fixed="right" prop="Oper" :label="$t('server.relayCdkeyOper')" width="60">
                <template #default="scope">
                    <div v-if="scope.row.Deleted == false">
                        <el-popconfirm :title="$t('server.relayCdkeyDelConfirm')" @confirm="handleDel(scope.row)">
                            <template #reference>
                                <el-button type="danger" size="small">
                                    <el-icon><Delete /></el-icon>
                                </el-button>
                            </template>
                        </el-popconfirm>
                    </div>
                </template>
            </el-table-column>
        </el-table>
        <div class="t-c">
            <div class="page">
                <el-pagination small background layout="prev, pager, next" 
                    :page-size="state.page.Size" 
                    :total="state.list.Count" 
                    :pager-count="5"
                    :current-page="state.page.Page" @current-change="handlePageChange" />
            </div>
        </div>
    </div>
    </el-dialog>
    <Add v-if="state.showAdd" v-model="state.showAdd" @success="handleSearch"></Add>
    <Test v-if="state.showTest" v-model="state.showTest"></Test>
</template>

<script>
import { injectGlobalData } from '@/provide';
import { onMounted, reactive, watch } from 'vue'
import { Delete,Plus,Search,Warning } from '@element-plus/icons-vue';
import { useI18n } from 'vue-i18n';
import { relayCdkeyDel,relayCdkeyPage } from '@/apis/relay';
import Flags from './Flags.vue';
import Add from './Add.vue';
import Test from './Test.vue';
export default {
    props: ['modelValue'],
    emits: ['update:modelValue'],
    components:{Delete,Plus,Search ,Flags,Add,Test,Warning},
    setup(props,{emit}) {
        const {t} = useI18n();
        const globalData = injectGlobalData();
        const state = reactive({
            page:{
                Page:1,
                Size:10,
                Order:'',
                Sort:'',
                UserId:'',
                OrderId:'',
                Contact:'',
                Remark:'',
                Flag:0
            },
            list:{
                Page:1,
                Size:15,
                Count:0,
                List:[]
            },
            show:true,
            showAdd:false,
            showTest:false
        });
        watch(() => state.show, (val) => {
            if (!val) {
                setTimeout(() => {
                    emit('update:modelValue', val);
                }, 300);
            }
        });
        const parseSpeed = (num) => {
            let index = 0;
            while (num >= 1024) {
                num /= 1024;
                index++;
            }
            return `${(num*1.0).toFixed(2)}${['B', 'KB', 'MB', 'GB', 'TB'][index]}`;
        }

        const handleFlagsChange = (flag)=>{
            state.page.Flag = flag;
            handleSearch();
        }
        const handleSearch = ()=>{
            relayCdkeyPage(state.page).then((res)=>{
                state.list = res;
            }).catch(()=>{})
        }
        const handlePageChange = (p)=>{
            state.page.Page = p;
            handleSearch();
        }
        const handleSort = (a)=>{
            state.page.Order = a.prop;
            state.page.Sort = {'ascending':'asc','descending':'desc'}[a.order];
            handleSearch();
        }

        const handleDel = (row)=>{
            relayCdkeyDel(row.Id).then((res)=>{
                handleSearch();
            }).catch(()=>{})
        }

        onMounted(()=>{
            handleSearch();
        })

        return {state,parseSpeed,handleSort,handleFlagsChange,handleSearch,handlePageChange,handleDel}
    }
}
</script>
<style lang="stylus" scoped>
.head{
    .search{
        &>div{
            margin-right:1rem;
        }
    }
}
.page{
    padding:2rem 0;
    display:inline-block;
}
.el-form-item{margin-bottom:1rem}
.el-input-number--small{width:10rem !important}
</style>
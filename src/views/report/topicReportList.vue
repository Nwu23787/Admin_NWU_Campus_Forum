<!-- 话题举报列表 -->
<template>
    <div class="main topicReportModule">
        <div class="navbar">
            <el-button type="primary" plain @click="$router.push({path: '/admin/control/topic/manage/view', query:{ visible:($route.query.visible != undefined ? $route.query.visible:''),topicView_beforeUrl:($route.query.topicView_beforeUrl != undefined ? $route.query.topicView_beforeUrl:''),topicId :$route.query.topicId, commentId:($route.query.commentId != undefined ? $route.query.commentId:''), page:($route.query.topicPage != undefined ? $route.query.topicPage:'')}})">返回</el-button>
        
            <el-button type="primary" plain v-if="verifyPermissionMenu('/control/report/manage?method=reduction*','post')" @click="reductionReport($event)">还原</el-button>
            <el-button type="primary" plain v-if="verifyPermissionMenu('/control/report/manage?method=delete*','post')" @click="deleteReport($event,undefined)">批量删除</el-button>
        </div>
        <div class="headInfo">
            <div class="title">{{state.currentTopic.title}}</div>
        </div>
        <div class="data-table" >
            <el-table ref="multipleTable" :data="state.tableData" tooltip-effect="dark" style="width: 100%" @selection-change="handleSelectionChange"  stripe empty-text="没有内容">
                <el-table-column type="selection" ></el-table-column>
                <el-table-column prop="reason" label="理由" align="center" ></el-table-column>
                <el-table-column prop="reportTypeName" label="分类" align="center" ></el-table-column>
                <el-table-column label="图片" align="center">
                    <template #default="scope">
                        <span v-for="imageInfo in scope.row.imageInfoList" class="image-preview">
                             <!-- 本地存储 -->
                            <el-image v-if="store_fileStorageSystem == 0" preview-teleported style="width: 60px; height: 60px" fit="cover" :src="imageInfo.path+imageInfo.name" :preview-src-list="[imageInfo.path+imageInfo.name]" hide-on-click-modal ></el-image>
                            <!-- SeaweedFS存储 使用nginx image_filter的缩略图处理功能 -->
                            <el-image v-if="store_fileStorageSystem == 10" preview-teleported style="width: 60px; height: 60px" fit="cover" :src="imageInfo.path+imageInfo.name+'?width=60'" :preview-src-list="[imageInfo.path+imageInfo.name]" hide-on-click-modal ></el-image>
                            <!--  MinIO存储 使用nginx image_filter的缩略图处理功能 -->
                            <el-image v-if="store_fileStorageSystem == 20" preview-teleported style="width: 60px; height: 60px" fit="cover" :src="imageInfo.path+imageInfo.name+'?width=60'" :preview-src-list="[imageInfo.path+imageInfo.name]" hide-on-click-modal ></el-image>
                            <!-- 阿里云OSS存储 -->
                            <el-image v-if="store_fileStorageSystem == 30" preview-teleported style="width: 60px; height: 60px" fit="cover" :src="imageInfo.path+imageInfo.name+'?x-oss-process=image/resize,w_60'" :preview-src-list="[imageInfo.path+imageInfo.name]" hide-on-click-modal ></el-image>
                        </span>
                    </template>
                </el-table-column>
                <el-table-column label="会员" align="center" min-width="120">
                    <template #default="scope">
                        <el-popover effect="light" trigger="hover" placement="top">
                            <template #default>
                                <p v-if="scope.row.isStaff == false">呢称: {{scope.row.nickname}}</p>
                                <p>账号: {{scope.row.account}}</p>
                            </template>
                            <template #reference>
                                
                               
                                <div class="avatar-wrapper"  @click="$router.push({path: '/admin/control/user/manage/show', query:{ id : scope.row.userId}})">
                                    <div class="avatar-badge" v-if="scope.row.avatarName == null || scope.row.avatarName == ''">
                                        <el-avatar :size="48" :icon="UserFilled"></el-avatar>
                                    </div>
                                    <div class="avatar-badge" v-if="scope.row.avatarName != null && scope.row.avatarName != ''">
                                        <el-avatar :size="48" :src="scope.row.avatarPath+'100x100/'+scope.row.avatarName"></el-avatar>
                                    </div>
                                    
                                    <div class="avatar-text">{{scope.row.account}}</div>
                                </div>
                            </template>
                        </el-popover>
                    </template>
                </el-table-column>
                <el-table-column align="center" width="120">
                    <template #header>
                        被举报内容
                    </template>
                    <template #default="scope">
                        <router-link v-if="scope.row.module == 10" tag="div" class="link" :to="{path: '/admin/control/topic/manage/view', query:{ topicId : scope.row.parameterId, reportModule:scope.row.module}}">查看话题</router-link>
                        <router-link v-if="scope.row.module == 20" tag="div" class="link" :to="{path: '/admin/control/topic/manage/view', query:{ topicId : scope.row.extraParameterId,commentId:scope.row.parameterId, reportModule:scope.row.module}}">查看评论</router-link>
                        <router-link v-if="scope.row.module == 30" tag="div" class="link" :to="{path: '/admin/control/topic/manage/view', query:{ topicId : scope.row.extraParameterId.split('-')[0], commentId:scope.row.extraParameterId.split('-')[1], replyId:scope.row.parameterId, reportModule:scope.row.module}}">查看评论回复</router-link>
                        <router-link v-if="scope.row.module == 40" tag="div" class="link" :to="{path: '/admin/control/question/manage/view', query:{ questionId : scope.row.parameterId, reportModule:scope.row.module}}">查看问题</router-link>
                        <router-link v-if="scope.row.module == 50" tag="div" class="link" :to="{path: '/admin/control/question/manage/view', query:{ questionId : scope.row.extraParameterId,answerId:scope.row.parameterId, reportModule:scope.row.module}}">查看答案</router-link>
                        <router-link v-if="scope.row.module == 60" tag="div" class="link" :to="{path: '/admin/control/question/manage/view', query:{ questionId : scope.row.extraParameterId.split('-')[0], answerId:scope.row.extraParameterId.split('-')[1], replyId:scope.row.parameterId, reportModule:scope.row.module}}">查看答案回复</router-link>
                    </template>
                </el-table-column>
                <el-table-column label="状态" align="center" width="160">
                    <template #default="scope">
                        <el-tag effect="dark"  v-if="scope.row.status==10" class="tag-wrapper">待处理</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==40" type="warning" class="tag-wrapper">投诉失败</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==50" type="success" class="tag-wrapper">投诉成功</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==1010" type="danger" class="tag-wrapper">待处理【用户删除】</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==1040" type="danger" class="tag-wrapper">投诉失败【用户删除】</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==1050" type="danger" class="tag-wrapper">投诉成功【用户删除】</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==100010" type="danger" class="tag-wrapper">待处理【员工删除】</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==100040" type="danger" class="tag-wrapper">投诉失败【员工删除】</el-tag>
                        <el-tag effect="dark"  v-if="scope.row.status==100050" type="danger" class="tag-wrapper">投诉成功【员工删除】</el-tag>
                    </template>
                </el-table-column>
                <el-table-column label="处理结果" align="center" width="180">
                    <template #default="scope">
                        <div>{{scope.row.processResult}}</div>
                    </template>
                </el-table-column>
                <el-table-column label="举报时间 / 处理完成时间" align="center" width="180">
                    <template #default="scope">
                        <div>{{scope.row.postTime}}</div>
                        <div v-if="scope.row.processCompleteTime" class="processCompleteTime">{{scope.row.processCompleteTime}}</div>
                        
                    </template>
                </el-table-column>
                <el-table-column prop="remark" label="备注" align="center" width="100"></el-table-column>
                <el-table-column label="IP归属地" align="center" width="100">
                    <template #header>
                        <el-popover effect="light" trigger="hover" placement="top" width="400">
                            <template #default>
                                <el-switch v-model="state.ip_text" inactive-text="IP" active-text="本列单元格显示IP"></el-switch><div><br></div>
                                <el-switch v-model="state.ipAddress_text" inactive-text="IP归属地" active-text="本列单元格显示IP归属地"></el-switch>
                            </template>
                            <template #reference>
                                <div class="header-arrow">IP归属地<el-icon class="header-arrow-icon"><ArrowDown /></el-icon></div>
                            </template>
                        </el-popover>
                        
                    </template>
                    <template #default="scope">
                        <div v-if="state.ip_text == true">
                            {{scope.row.ip}}
                        </div>
                        <div v-if="state.ipAddress_text == true">
                            {{scope.row.ipAddress}}
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="staffAccount" label="处理员工" align="center" width="100"></el-table-column>
                <el-table-column label="操作" align="center" fixed="right" width="200">
                    <template #default="scope">
                        <div class="button-group-wrapper">
                            <el-button-group>
                                <el-button type="primary" :disabled="!verifyPermissionMenu('/control/report/manage?method=reportHandle*','get')" v-if="scope.row.status==10" @click="processUI(scope.row)">处理</el-button>
                                <el-button type="primary" :disabled="!verifyPermissionMenu('/control/report/manage?method=edit*','get')" @click="$router.push({path: '/admin/control/report/manage/edit', query:{ reportId : scope.row.id}})">修改</el-button>
                                <el-button type="primary" :disabled="!verifyPermissionMenu('/control/report/manage?method=delete*','post')" @click="deleteReport($event,scope.row)">删除</el-button>
                            </el-button-group>
                        </div>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination-wrapper" v-if="state.isShowPage">
                <el-pagination background  @current-change="page" :current-page="state.currentpage"  :page-size="state.maxresult" layout="total, prev, pager, next,jumper" :total="state.totalrecord"></el-pagination>
            </div>
            <div class="processModule">
            
                <el-dialog title="处理" width="60%" v-model="state.popup_process" @close="closeProcessWindow">
                    
                    <div class="data-form" >
                        <el-form label-width="130px"  @submit.native.prevent>
                            <el-form-item label="处理状态" >
                                <el-radio-group v-model="state.status" size="large">
                                    <el-radio-button :label="40">投诉失败</el-radio-button>
                                    <el-radio-button :label="50">投诉成功</el-radio-button>
                                </el-radio-group>
                            </el-form-item>
                            <el-form-item label="处理结果">
                                <el-input v-model="state.processResult" type="textarea" :autosize="{ minRows: 2, maxRows: 4}" ></el-input>
                            </el-form-item>
                            <el-form-item label="备注">
                                <el-input v-model="state.remark" type="textarea" :autosize="{ minRows: 2, maxRows: 4}" ></el-input>
                                <div class="form-help" >本内容不在前台显示</div>
                            </el-form-item>
                            
                            
                            <el-form-item>
                                <el-button type="primary" size="large" class="submitButton"  @click="submitForm" :disabled="state.submitForm_disabled">提交</el-button>
                            </el-form-item> 
                        </el-form>
                    </div>
                    
                </el-dialog>
            </div>
        </div>
    </div>
</template>
<script lang="ts" setup>
    import { ComponentInternalInstance, getCurrentInstance, onMounted, reactive, ref } from 'vue';
    import pinia from '@/store/store'
    import {useStore} from '@/store'
    import { storeToRefs } from 'pinia';
    import { AxiosResponse } from 'axios';
    import { useRouter } from 'vue-router';
    import { ElMessage, ElMessageBox, ElTable } from 'element-plus';
    import { processErrorInfo,verifyPermissionMenu } from '@/utils/tool';
    import { UserFilled } from '@element-plus/icons-vue'
    import { Topic } from '@/types';
    
    const store = useStore(pinia);
    const { proxy } = getCurrentInstance() as ComponentInternalInstance;
    const router = useRouter();
    const {fileStorageSystem:store_fileStorageSystem} = storeToRefs(store)

    const multipleTable = ref<InstanceType<typeof ElTable>>()

    const state = reactive({
        tableData: [],//表格内容

        currentTopic :{} as Topic,
        topicId:'',
        parameterId:'',
        module:0,

        multipleSelection: [],
        
        totalrecord : 0, //总记录数
        currentpage : 1, //当前页码
        totalpage : 1, //总页数
        maxresult: 12, //每页显示记录数
        isShowPage:false,//是否显示分页 maxresult没返回结果前就显示会导致分页栏显示页码错误

        ip_text :false,//IP文本
        ipAddress_text :true,//IP归属地文本


        popup_process:false,//是否弹出处理窗口
        submitForm_disabled:false,//提交按钮是否禁用
        reportId:'',//举报Id
        remark:'',//备注
        status:50,//处理状态
        processResult:'',//处理结果
        version:0,//版本号
    });

    const error = reactive({
    });

    //查询话题举报列表
    const queryTopicReportList = () => {
        //清空内容
        state.tableData = []; 


        proxy?.$axios({
            url: '/control/topicReport/list',
            method: 'get',
            params: {
                topicId :state.topicId,
                parameterId :state.parameterId,
                module :state.module,
                page :state.currentpage
            },
           // showLoading: false,//是否显示加载图标
            loadingMask:false,// 是否显示遮罩层
        })
        .then((response: AxiosResponse) => {
            if(response){
                const result: any = response.data;
                if(result){
                    let returnValue = JSON.parse(result);
                    if(returnValue.code === 200){//成功
                        let mapData = returnValue.data;
			    		for(let key in mapData){
			    			if(key == "currentTopic"){
			    				state.currentTopic = mapData[key];
			    				
			    			}else if(key == "pageView"){
			    				let pageView = mapData[key];
			    				let list = pageView.records;
					    		if(list != null && list.length >0){
					    			state.tableData = list;
					 
					    			state.totalrecord = parseInt(pageView.totalrecord);//服务器返回的long类型已转为String类型
					    			state.currentpage = pageView.currentpage;
									state.totalpage = parseInt(pageView.totalpage);//服务器返回的long类型已转为String类型
									state.maxresult = pageView.maxresult;
									state.isShowPage = true;//显示分页栏
					    		}
			    			}
			    		}
                    }else if(returnValue.code === 500){//错误
			    		//处理错误信息
                        processErrorInfo(returnValue.data as Map<string,string> , reactive({}),()=>{});

			    		
			    	}
                }
            }
        })
        .catch((error: any) =>{
            console.log(error);
        });
    }
    //分页
    const page = (page:number) => {
        //删除缓存
        store.setCacheNumber();
        router.push({
            path: '/admin/control/topicReport/list', 
            query:{ 
                visible:(router.currentRoute.value.query.visible != undefined ? router.currentRoute.value.query.visible:''),
                topicView_beforeUrl:(router.currentRoute.value.query.topicView_beforeUrl != undefined ?router.currentRoute.value.query.topicView_beforeUrl:''),
                topicId :state.topicId,
                commentId :(router.currentRoute.value.query.commentId != undefined ? router.currentRoute.value.query.commentId:''),
                topicPage :(router.currentRoute.value.query.topicPage != undefined ? router.currentRoute.value.query.topicPage:''),
                parameterId : state.parameterId,
                module: state.module,
                ip_text : String(state.ip_text),
                ipAddress_text : String(state.ipAddress_text),
                page : page
            }
        });
    }

    //处理多选
    const handleSelectionChange = (val:any) => {
        state.multipleSelection = val;
    }

    //处理UI
    const processUI = (row:any) => {
        state.popup_process=true;
	    state.reportId = row.id;
        state.version = row.version;
        //清空内容
        state.status = 50;
        state.processResult = '';
        state.remark = '';
    }
    //关闭处理弹出框
    const closeProcessWindow = () => {
        state.popup_process=false;
    }
    //提交表单
    const submitForm = () => {
        state.submitForm_disabled = true;

        //清空error的属性值
        Object.keys(error).map(key => {
            Object.assign(error, {[key] : ''});
        })

        let formData = new FormData();
        formData.append('reportId', state.reportId);

        formData.append('status', String(state.status));
        formData.append('processResult', state.processResult);
        formData.append('remark', state.remark);
        formData.append('version', String(state.version));
        


        proxy?.$axios({
            url: '/control/report/manage?method=reportHandle',
            method: 'post',
            data: formData,
            //showLoading: false,//是否显示加载图标
            loadingMask:false,// 是否显示遮罩层
        })
        .then((response: AxiosResponse) => {
            if(response){
                const result: any = response.data;
			    if(result){
                    let returnValue = JSON.parse(result);
			    	if(returnValue.code === 200){//成功
                        ElMessage({
                            showClose: true,
                            message: '提交成功',
                            type: 'success',
                            onClose: ()=>{
                                
                            }
                        })
			    		//删除缓存
                        store.setCacheNumber();
			    		closeProcessWindow()
                        queryTopicReportList();
			    	}else if(returnValue.code === 500){//错误
			    		//处理错误信息
                        processErrorInfo(returnValue.data as Map<string,string> , error,()=>{});

			    		
			    	}
                }
                state.submitForm_disabled = false;//提交按钮disabled状态
            }
        })
        .catch((error: any) =>{
            console.log(error);
            state.submitForm_disabled = false;//提交按钮disabled状态
        });
    }

    //还原举报
    const reductionReport = (event:any) => {
        //强制失去焦点
        let target = event.target;
        // 根据button组件内容 里面包括一个span标签，如果设置icon，则还包括一个i标签，其他情况请自行观察。
        // 所以，在我们点击到button组件上的文字也就是span标签上时，直接执行e.target.blur()不会生效，所以要加一层判断。
        if(target.nodeName == 'SPAN' || target.nodeName == 'I'){
            target = event.target.parentNode;
        }
        target.blur();

        if(state.multipleSelection.length <1){
            ElMessage({
                //duration :0,
                showClose: true,
                message: '至少要选择一行数据',
                type: 'error',
            })
            return;
        }

        ElMessageBox.confirm('此操作将还原该举报, 是否继续?',{
            // type: 'warning',
            cancelButtonText: "取消",
            confirmButtonText: '确定'
        })
        .then(() => {

            let formData = new FormData();
            for(let i=0; i<state.multipleSelection.length; i++){
                let rowData:any = state.multipleSelection[i];
                formData.append('reportId', rowData.id);
            }
        

            proxy?.$axios({
                url: '/control/report/manage?method=reduction',
                method: 'post',
                data: formData
            })
            .then((response: AxiosResponse) => {
                if(response){

                    const result: any = response.data;
                
                    if(result){
				    	
				    	let returnValue = JSON.parse(result);
				    	if(returnValue.code === 200){//成功
				    		//删除缓存
                            store.setCacheNumber();
                            ElMessage({
                                showClose: true,
                                message: '还原成功',
                                type: 'success',
                                onClose: ()=>{
                                    
                                }
                            })
				    		queryTopicReportList();
				    	}else if(returnValue.code === 500){//错误
				    		//处理错误信息
                            processErrorInfo(returnValue.data as Map<string,string> , reactive({}),()=>{});
				    		
				    		
				    	}
				    }
                }
            })
            .catch((error: any) =>{
                console.log(error);
            });

        })
        .catch(() => {
            //取消
        })

    }
    
    //删除举报
    const deleteReport = (event:any,row:any) => {
        //强制失去焦点
        let target = event.target;
        // 根据button组件内容 里面包括一个span标签，如果设置icon，则还包括一个i标签，其他情况请自行观察。
        // 所以，在我们点击到button组件上的文字也就是span标签上时，直接执行e.target.blur()不会生效，所以要加一层判断。
        if(target.nodeName == 'SPAN' || target.nodeName == 'I'){
            target = event.target.parentNode;
            }
        target.blur();

        
        if (row) {//选中行
            multipleTable.value!.toggleRowSelection(row,true);
        }
        
        if(state.multipleSelection.length <1){
            ElMessage({
                //duration :0,
                showClose: true,
                message: '至少要选择一行数据',
                type: 'error',
            })
            return;
        }

       
        ElMessageBox.confirm('此操作将删除该项, 是否继续?',{
            // type: 'warning',
            cancelButtonText: "取消",
            confirmButtonText: '确定'
        })
        .then(() => {

            let formData = new FormData();
		    	
            
            for(let i=0; i<state.multipleSelection.length; i++){
                let rowData:any = state.multipleSelection[i];
                formData.append('reportId', rowData.id);
            }

            proxy?.$axios({
                url: '/control/report/manage?method=delete',
                method: 'post',
                data: formData
            })
            .then((response: AxiosResponse) => {
                if(response){

                    const result: any = response.data;
                
                    if(result){
				    	
				    	let returnValue = JSON.parse(result);
				    	if(returnValue.code === 200){//成功
                            ElMessage({
                                showClose: true,
                                message: '删除成功',
                                type: 'success',
                                onClose: ()=>{
                                    
                                }
                            })
				    		queryTopicReportList();
				    	}else if(returnValue.code === 500){//错误
				    		
                            //处理错误信息
                            processErrorInfo(returnValue.data as Map<string,string> , reactive({}),()=>{});

				    		
				    	}
				    }
                }
            })
            .catch((error: any) =>{
                console.log(error);
            });

        })
        .catch(() => {
            //取消
            //取消选中行
	        multipleTable.value!.toggleRowSelection(row,false);
        })
    }


    onMounted(() => {
        //设置缓存
        store.setCacheComponents(String(router.currentRoute.value.name))

        state.topicId = router.currentRoute.value.query.topicId != undefined ? router.currentRoute.value.query.topicId as string :'';
        state.parameterId = router.currentRoute.value.query.parameterId != undefined ? router.currentRoute.value.query.parameterId as string :'';
        state.module = (router.currentRoute.value.query.module != undefined && router.currentRoute.value.query.module != '') ? parseInt(router.currentRoute.value.query.module as string) :0;
       
        state.ip_text = router.currentRoute.value.query.ip_text != undefined ? String(router.currentRoute.value.query.ip_text).toLowerCase() === 'true' :false;
        
        state.ipAddress_text = router.currentRoute.value.query.ipAddress_text != undefined ? String(router.currentRoute.value.query.ipAddress_text).toLowerCase() === 'true' :true;
      
        state.currentpage = (router.currentRoute.value.query.page != undefined && router.currentRoute.value.query.page != '') ? parseInt(router.currentRoute.value.query.page as string) :1;
        


        queryTopicReportList();
    }) 

</script>
<style scoped lang="scss">

.topicReportModule{
    .headInfo{
        margin-top: 6px;
        margin-left:8px;
        margin-right:8px;
        border-bottom:1px solid #eee;
        padding-bottom:4px;
        margin-bottom:5px;
        text-align: center;
        .title{
            margin-left:4px;
            font-size: 16px;
            line-height:26px;
            color:#909399;
            padding:6px 0;
        }
    }
}
.data-table{
    .processCompleteTime{
        color: #67C23A;
    }
    .staffAccount{
        color: #67C23A;
    }
}
</style>

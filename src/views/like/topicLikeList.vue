<!-- 话题点赞列表 -->
<template>
    <div class="main likeModule">
        <div class="navbar">
            <el-button type="primary" plain @click="$router.push({path: '/admin/control/topic/manage/view', query:{ visible:($route.query.visible != undefined ? $route.query.visible:''),topicView_beforeUrl:($route.query.topicView_beforeUrl != undefined ? $route.query.topicView_beforeUrl:''),topicId :$route.query.topicId, commentId:($route.query.commentId != undefined ? $route.query.commentId:''),itemId:$route.query.itemId,likeModule:$route.query.likeModule, page:($route.query.topicPage != undefined ? $route.query.topicPage:'')}})">返回</el-button>
        </div>
        <div class="headInfo">
            <div class="title">{{state.currentTopic.title}}</div>
        </div>
        <div class="data-table" >
            <el-table ref="multipleTable" :data="state.tableData" tooltip-effect="dark" style="width: 100%" stripe empty-text="没有内容">
                <el-table-column label="用户" align="center" min-width="120">
                    <template #default="scope">
                        <el-popover effect="light" trigger="hover" placement="top">
                            <template #default>
                                <p v-if="scope.row.nickname != null &&  scope.row.nickname != ''">呢称: {{scope.row.nickname}}</p>
                                <p>账号: {{scope.row.account}}</p>
                            </template>
                            <template #reference>
                                <div class="avatar-wrapper" >
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
                <el-table-column prop="addtime" label="加入时间" align="center" width="170"></el-table-column>
            </el-table>
            <div class="pagination-wrapper" v-if="state.isShowPage">
                <el-pagination background  @current-change="page" :current-page="state.currentpage"  :page-size="state.maxresult" layout="total, prev, pager, next,jumper" :total="state.totalrecord"></el-pagination>
            </div>
        </div>
    </div>
</template>
<script lang="ts" setup>
    import { ComponentInternalInstance, getCurrentInstance, onMounted, reactive, ref } from 'vue';
    import pinia from '@/store/store'
    import {useStore} from '@/store'
    import { AxiosResponse } from 'axios';
    import { useRouter } from 'vue-router';
    import { ElTable } from 'element-plus';
    import { UserFilled } from '@element-plus/icons-vue'
    import { Topic } from '@/types';

    const store = useStore(pinia);
    const { proxy } = getCurrentInstance() as ComponentInternalInstance;
    const router = useRouter();

    const multipleTable = ref<InstanceType<typeof ElTable>>()

    const state = reactive({
        tableData: [],//表格内容
        multipleSelection: [],

        currentTopic :{} as Topic,
        topicId:'',
        id :'',
        
        itemId :'',//项目Id
        likeModule :10,//模块


        userName :'',//用户名称
        
        totalrecord : 0, //总记录数
        currentpage : 1, //当前页码
        totalpage : 1, //总页数
        maxresult: 12, //每页显示记录数
        isShowPage:false,//是否显示分页 maxresult没返回结果前就显示会导致分页栏显示页码错误
    });

    //查询点赞
    const queryLikeList = () => {
        //清空内容
        state.tableData = []; 


        proxy?.$axios({
            url: '/control/topicLike/list',
            method: 'get',
            params: {
                topicId :state.topicId,
                itemId :state.itemId,
                likeModule :state.likeModule,
                id :state.id,
                userName :state.userName,
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
            path: '/admin/control/topicLike/list', 
            query:{ 
                visible:(router.currentRoute.value.query.visible != undefined ? router.currentRoute.value.query.visible:''),
                topicView_beforeUrl:(router.currentRoute.value.query.topicView_beforeUrl != undefined ?router.currentRoute.value.query.topicView_beforeUrl:''),
                topicId :state.topicId,
                commentId :(router.currentRoute.value.query.commentId != undefined ? router.currentRoute.value.query.commentId:''),
                itemId : (router.currentRoute.value.query.itemId != undefined ? router.currentRoute.value.query.itemId:''),
                likeModule : (router.currentRoute.value.query.likeModule != undefined ? router.currentRoute.value.query.likeModule:''),
                topicPage :(router.currentRoute.value.query.topicPage != undefined ? router.currentRoute.value.query.topicPage:''),
                page : page
            }
        });
    }

    onMounted(() => {
        //设置缓存
        store.setCacheComponents(String(router.currentRoute.value.name))

          
        state.currentpage = (router.currentRoute.value.query.page != undefined && router.currentRoute.value.query.page != '') ? parseInt(router.currentRoute.value.query.page as string) :1;
        




        if(router.currentRoute.value.query.topicId != undefined && router.currentRoute.value.query.topicId != ''){
			state.topicId = router.currentRoute.value.query.topicId as string;
			
		}
		if(router.currentRoute.value.query.id != undefined && router.currentRoute.value.query.id != ''){
			state.id = router.currentRoute.value.query.id as string;
			
		}

        if(router.currentRoute.value.query.itemId != undefined && router.currentRoute.value.query.itemId != ''){
			state.itemId = router.currentRoute.value.query.itemId as string;
			
		}
        if(router.currentRoute.value.query.likeModule != undefined && router.currentRoute.value.query.likeModule != ''){
			state.likeModule = parseInt(router.currentRoute.value.query.likeModule as string);
			
		}



		if(router.currentRoute.value.query.userName != undefined && router.currentRoute.value.query.userName != ''){
			state.userName = decodeURIComponent(router.currentRoute.value.query.userName as string);
			
		}
		queryLikeList();
    }) 

</script>
<style scoped lang="scss">
/* 话题点赞列表 */
.likeModule{
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
</style>
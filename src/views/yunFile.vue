<template>
    <div ref="wrap">
        <!-- 云盘 -->
        <div class="yun-file">
            <div class="pb35 mt20">
                <div class="yun-file-title flex">
                    <text class="f28 fw5 c0">{{i18n.FileIShare}}</text>
                    <text class="f24 c153 fw4 pl20 pt10 pb10" @click="yunFileMoreEvent">{{i18n.All}}</text>
                </div>
                <div class="mt36 prl20 file-height">
                    <div class="flex" v-if='isShowBM'>
                        <scroller v-if='yunFileByMeArr.length != 0' class="yun-scroller flex-dr" show-scrollbar='false'
                            scroll-direction="horizontal">
                            <div class="flex-ac file-name" v-if='item.isExitDoc' v-for="(item, index) in yunFileByMeArr" :key='index' @click='yunFileUserEvent(item.id)'>
                                <bui-image :src="item.image" width="52px" height="52px" @click='yunFileUserEvent(item.id)'></bui-image>
                                <div class="flex-jc flex-ac" style="width: 153px">
                                    <text class="f24 c51 fw4 mt17 lines2">{{item.name}}</text>
                                </div>
                            </div>
                        </scroller>
                        <div class="no-file flex-ac flex-jc" v-if='yunFileByMeArr.length == 0'>
                            <div class="flex-dr">
                                <bui-image src="/image/sleep1.png" width="42px" height="39px"></bui-image>
                                <text class="f26 c51 fw4 pl15 center-height ">{{isErrorBM?i18n.NoneData:i18n.ErrorLoadData}}</text>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    const link = weex.requireModule("LinkModule");
    const linkapi = require("linkapi");
    const dom = weex.requireModule('dom');
    export default {
        data() {
            return {
                yunFileByMeArr: [],
                isErrorBM: true,
                isShowBM: false,
                channel: new BroadcastChannel('WidgetsMessage'),
                i18n: ''
            }
        },
        methods: {
            yunFileMoreEvent() {
                link.launchLinkService(['[OpenBuiltIn] \n key=ShareByMeList'], (res) => { }, (err) => { });
            },
            yunFileUserEvent(id) {
                link.launchLinkService(['[OpenBuiltIn] \n key=DiskDetail \n diskId=' + id], (res) => { }, (err) => { });
            },
            getFileImages(ext) {
                let fileImageTypes = {
                    'excel': ['xls', 'xlsx'],
                    'music': ['mp3', 'wma', 'wav', 'mod', 'ogg', 'm4a'],
                    'pdf': ['pdf'],
                    'photo': ['bmp', 'gif', 'jpeg', 'jpg', 'svg', 'png', 'psd'],
                    'ppt': ['ppt', 'pptx'],
                    'txt': ['txt', 'key'],
                    'video': ['rm', 'rmvb', 'wmv', 'avi', 'mp4', '3gp', 'mkv', 'flv', 'mov', 'mpg'],
                    'word': ['doc', 'docx', 'wps'],
                    'zip': ['zip', 'rar', '7z'],
                    'unknow': ['file'],
                    'folder2': ['folder']
                }
                let fileImages = {};
                let fileTypeImages = {};
                for (let fext in fileImageTypes) {
                    fileImages[fext] = '/image/' + fext + '.png';
                    let arr = fileImageTypes[fext];
                    if (arr.length > 0) {
                        for (let i = 0; i < arr.length; i++) {
                            fileTypeImages[arr[i]] = fext;
                        }
                    }
                }
                let type = fileTypeImages[ext];
                if (type) {
                    return fileImages[type];
                } else {
                    return fileImages['unknow'];
                }
            },
            getYunFile() {
                link.getServerConfigs([], (params) => {
                    link.getLoginInfo([], (user) => {
                        let objDataBy = {
                            by: user.userId,
                            to: '',
                            scope: ''
                        }
                        let url = params.diskUrl ? params.diskUrl + '/' : params.diskUri
                        linkapi.get({
                            url: url + 'openapi//file/share/list',
                            data: objDataBy
                        }).then((res) => {
                            this.broadcastWidgetHeight()
                            this.isShowBM = true
                            this.isErrorBM = true
                            let fileArr = []
                            for (let index = 0, resLength = res.rows.length; index < resLength; index++) {
                                let fileObj = {}
                                const element = res.rows[index];
                                fileObj['name'] = element.name
                                fileObj['id'] = element.fileId
                                if (element.type == 'D') {
                                    fileObj['isExitDoc'] = false
                                    fileObj['image'] = '/image/folder2.png'
                                } else {
                                    fileObj['isExitDoc'] = true
                                    fileObj['image'] = this.getFileImages(element.extension)
                                }
                                fileArr.push(fileObj)
                            }
                            this.yunFileByMeArr = fileArr
                        }, (err) => {
                            this.error()
                        })
                    }, (err) => {
                        this.error()
                    })
                }, () => {
                    this.error()
                });
            },
            error() {
                this.isShowBM = true
                this.isErrorBM = false
                this.broadcastWidgetHeight()
            },
            broadcastWidgetHeight() {
                let _params = this.$getPageParams();
                setTimeout(() => {
					dom.getComponentRect(this.$refs.wrap, (ret) => {
						this.channel.postMessage({
							widgetHeight: ret.size.height,
							id: _params.id
						});
					});
				}, 200)
            }
        },
        created() {
            this.$fixViewport();
            linkapi.getLanguage((res) => {
                this.i18n = this.$window[res]
            })
        },
        mounted() {
            this.channel.onmessage = (event) => {
                if (event.data.action === 'RefreshData') {
                    this.getYunFile()
                }
            }
            this.getYunFile()
        }
    }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
    .yun-file {
        background-color: #fff;
    }

    .yun-file-title {
        height: 40px;
        margin: 18px 23px 36px 25px;
    }

    .file-name {
        margin-right: 15px;
        width: 166px;
    }

    .yun-scroller {
        flex: 1;
        height: 140px;
    }

    .no-file {
        height: 166px;
        flex: 1;
    }

    .center-height {
        line-height: 40px;
    }
</style>
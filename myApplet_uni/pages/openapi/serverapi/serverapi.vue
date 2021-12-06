<template>
    <view class="container">
        <view class="list">
            <view class="list-item" @tap="getSubscribeMessageTemplate">
                <text>获取订阅消息模板 ID</text>
            </view>
            <view class="list-item" v-if="templateId">
                <text class="request-text">模板 ID：{{ templateId }}</text>
            </view>
        </view>

        <view class="list">
            <view class="list-item" @tap="requestSubscribeMessage">
                <text>获取下发权限</text>
            </view>
            <view class="list-item" v-if="requestSubscribeMessageResult">
                <text class="request-text">获取权限结果：{{ requestSubscribeMessageResult }}</text>
            </view>
        </view>

        <view class="list">
            <view class="list-item" @tap="sendSubscribeMessage">
                <text>发送订阅消息</text>
            </view>
            <view class="list-item" v-if="subscribeMessageResult">
                <text class="request-text">调用结果：{{ subscribeMessageResult }}</text>
            </view>
        </view>

        <view class="guide">
            <text class="headline">测试须知</text>
            <text class="p">1. 需先到小程序管理后台，进入订阅消息管理</text>
            <text class="p">2. 在订阅消息管理、公共模板库中添加一个模板</text>
            <text class="p">3. 添加完成后在我的模板中点开模板详情</text>
            <text class="p">4. 根据模板详情修改 openapi 云函数 index.js 中的相应位置</text>
            <text class="p">5. 上传 cloudfunctions 目录下的 openapi 云函数</text>
            <text class="p">6. 需在手机上预览测试，工具中无效</text>
            <text class="p">7. 依次点击获取模板、获取下发权限、发送订阅消息</text>
            <text class="p">8. 调用成功后返回到微信主界面查看收到的模板消息</text>
        </view>

        <view class="list">
            <view class="list-item" @tap="onGetWXACode">
                <text>获取小程序码</text>
            </view>
            <view class="list-item" v-if="wxacodeResult">
                <text class="request-text">{{ wxacodeResult }}</text>
                <text class="request-text" v-if="showClearWXACodeCache" @tap="clearWXACodeCache">清除缓存</text>
            </view>
        </view>

        <view class="guide">
            <text class="headline">测试须知</text>
            <text class="p">1. 需上传 cloudfunctions 目录下的 openapi 云函数</text>
            <text class="p">2. 云函数中获取图片后会上传至存储空间并返回至小程序使用和缓存</text>
            <text class="p">3. 云存储需设置为公有读</text>
        </view>

        <view class="guide">
            <image :src="wxacodeSrc" mode="aspectFit"></image>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            templateId: '',
            subscribeMessageResult: '',
            requestSubscribeMessageResult: '',
            wxacodeSrc: '',
            wxacodeResult: '',
            showClearWXACodeCache: false,
            templateMessageResult: ''
        };
    },
    methods: {
        async getSubscribeMessageTemplate() {
            try {
                const { result } = await wx.cloud.callFunction({
                    name: 'openapi',
                    data: {
                        action: 'requestSubscribeMessage'
                    }
                });
                const templateId = result;
                console.warn('[云函数] [openapi] 获取订阅消息模板 调用成功：', templateId);
                this.setData({
                    templateId
                });
            } catch (err) {
                uni.showToast({
                    icon: 'none',
                    title: '调用失败'
                });
                console.error('[云函数] [openapi] 获取订阅消息模板 调用失败：', err);
            }
        },

        async requestSubscribeMessage() {
            const templateId = this.templateId;

            if (!templateId) {
                uni.showModal({
                    title: '发送失败',
                    content: '请先获取模板 ID',
                    showCancel: false
                });
            }

            uni.requestSubscribeMessage({
                tmplIds: [templateId],
                success: (res) => {
                    if (res[templateId] === 'accept') {
                        this.setData({
                            requestSubscribeMessageResult: '成功'
                        });
                    } else {
                        this.setData({
                            requestSubscribeMessageResult: `失败（${res[templateId]}）`
                        });
                    }
                },
                fail: (err) => {
                    this.setData({
                        requestSubscribeMessageResult: `失败（${JSON.stringify(err)}）`
                    });
                }
            });
        },

        sendSubscribeMessage(e) {
            this.setData({
                subscribeMessageResult: ''
            });
            wx.cloud.callFunction({
                name: 'openapi',
                data: {
                    action: 'sendSubscribeMessage',
                    templateId: this.templateId
                },
                success: (res) => {
                    console.warn('[云函数] [openapi] subscribeMessage.send 调用成功：', res);
                    uni.showModal({
                        title: '发送成功',
                        content: '请返回微信主界面查看',
                        showCancel: false
                    });
                    uni.showToast({
                        title: '发送成功，请返回微信主界面查看'
                    });
                    this.setData({
                        subscribeMessageResult: JSON.stringify(res.result)
                    });
                },
                fail: (err) => {
                    uni.showToast({
                        icon: 'none',
                        title: '调用失败'
                    });
                    console.error('[云函数] [openapi] subscribeMessage.send 调用失败：', err);
                }
            });
        },

        submitSubscribeMessageForm(e) {
            this.setData({
                subscribeMessageResult: ''
            });
            wx.cloud.callFunction({
                name: 'openapi',
                data: {
                    action: 'sendSubscribeMessage',
                    formId: e.detail.formId
                },
                success: (res) => {
                    console.warn('[云函数] [openapi] subscribeMessage.send 调用成功：', res);
                    uni.showModal({
                        title: '发送成功',
                        content: '请返回微信主界面查看',
                        showCancel: false
                    });
                    uni.showToast({
                        title: '发送成功，请返回微信主界面查看'
                    });
                    this.setData({
                        templateMessageResult: JSON.stringify(res.result)
                    });
                },
                fail: (err) => {
                    uni.showToast({
                        icon: 'none',
                        title: '调用失败'
                    });
                    console.error('[云函数] [openapi] templateMessage.send 调用失败：', err);
                }
            });
        },

        onGetWXACode() {
            this.setData({
                wxacodeSrc: '',
                wxacodeResult: '',
                showClearWXACodeCache: false
            }); // 此处为演示，将使用 localStorage 缓存，正常开发中文件 ID 应存在数据库中

            const fileID = uni.getStorageSync('wxacodeCloudID');

            if (fileID) {
                // 有云文件 ID 缓存，直接使用该 ID
                // 如需清除缓存，选择菜单栏中的 “工具 -> 清除缓存 -> 清除数据缓存”，或在 Storage 面板中删掉相应的 key
                this.setData({
                    wxacodeSrc: fileID,
                    wxacodeResult: `从本地缓存中取得了小程序码的云文件 ID`,
                    showClearWXACodeCache: true
                });
                console.log(`从本地缓存中取得了小程序码的云文件 ID：${fileID}`);
            } else {
                wx.cloud.callFunction({
                    name: 'openapi',
                    data: {
                        action: 'getWXACode'
                    },
                    success: (res) => {
                        console.warn('[云函数] [openapi] wxacode.get 调用成功：', res);
                        uni.showToast({
                            title: '调用成功'
                        });
                        this.setData({
                            wxacodeSrc: res.result,
                            wxacodeResult: `云函数获取二维码成功`,
                            showClearWXACodeCache: true
                        });
                        uni.setStorageSync('wxacodeCloudID', res.result);
                    },
                    fail: (err) => {
                        uni.showToast({
                            icon: 'none',
                            title: '调用失败'
                        });
                        console.error('[云函数] [openapi] wxacode.get 调用失败：', err);
                    }
                });
            }
        },

        clearWXACodeCache() {
            uni.removeStorageSync('wxacodeCloudID');
            this.setData({
                wxacodeSrc: '',
                wxacodeResult: '',
                showClearWXACodeCache: false
            });
            uni.showToast({
                title: '清除成功'
            });
        }
    }
};
</script>
<style>
@import './serverapi.css';
</style>

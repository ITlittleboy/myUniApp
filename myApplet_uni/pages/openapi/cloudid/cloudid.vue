<template>
    <!--index.wxml-->
    <view class="container">
        <view class="guide">
            <text class="headline">开放数据调用</text>
            <text class="p">通过 cloudID 获取敏感开放数据有以下两种方式</text>
            <text class="p">1. 小程序端 callFunction 自动获取</text>
            <text class="p">2. 通过 wx-server-sdk 获取</text>
            <text class="p">以下分别先后展示这两种获取方式</text>
        </view>

        <view class="uploader">
            <button class="uploader-text" @tap="onGetWeRunData">getWeRunData 敏感数据获取</button>
        </view>

        <view class="guide">
            <text class="headline">测试须知</text>
            <text class="p">1. 公共库版本需大于 2.7.0</text>
            <text class="p">2. 请确保 echo 函数已上传</text>
        </view>

        <view class="guide" style="word-break: break-all">
            {{ weRunResult }}
        </view>

        <view class="uploader">
            <button class="uploader-text" open-type="getUserInfo" @getuserinfo="onGetUserInfo">getUserInfo 敏感数据获取</button>
        </view>

        <view class="guide">
            <text class="headline">测试须知</text>
            <text class="p">1. 公共库版本需大于 2.7.0</text>
            <text class="p">2. 请确保 openapi 函数已上传</text>
        </view>

        <view class="guide" style="word-break: break-all">
            {{ userInfoResult }}
        </view>
    </view>
</template>

<script>
// miniprogram/pages/openapi/cloudid/cloudid.js
export default {
    data() {
        return {
            weRunResult: '',
            userInfoResult: ''
        };
    },
    methods: {
        onGetWeRunData() {
            uni.getWeRunData({
                success: (res) => {
                    wx.cloud
                        .callFunction({
                            name: 'echo',
                            data: {
                                // info 字段在云函数 event 对象中会被自动替换为相应的敏感数据
                                info: wx.cloud.CloudID(res.cloudID)
                            }
                        })
                        .then((res) => {
                            console.log('[onGetWeRunData] 收到 echo 回包：', res);
                            this.setData({
                                weRunResult: JSON.stringify(res.result)
                            });
                            uni.showToast({
                                title: '敏感数据获取成功'
                            });
                        })
                        .catch((err) => {
                            console.log('[onGetWeRunData] 失败：', err);
                        });
                }
            });
        },

        onGetUserInfo(e) {
            console.log(e);
            wx.cloud
                .callFunction({
                    name: 'openapi',
                    data: {
                        action: 'getOpenData',
                        openData: {
                            list: [e.detail.cloudID]
                        }
                    }
                })
                .then((res) => {
                    console.log('[onGetUserInfo] 调用成功：', res);
                    this.setData({
                        userInfoResult: JSON.stringify(res.result)
                    });
                    uni.showToast({
                        title: '敏感数据获取成功'
                    });
                });
        }
    }
};
</script>
<style>
@import './cloudid.css';
</style>

<template>
    <!--index.wxml-->
    <view class="container">
        <!-- 用户 openid -->
        <view class="userinfo">
            <block v-if="canIUseOpenData" class="userinfo-opendata">
                <view class="userinfo-block-avatar" @tap="bindViewTap">
                    <open-data type="userAvatarUrl"></open-data>
                </view>
            </block>
            <block v-else-if="!hasUserInfo">
                <button v-if="canIUseGetUserProfile" @tap="getUserProfile" class="userinfo-avatar" :style="'background-image: url(' + avatarUrl + ')'" size="default"></button>
                <button
                    v-else
                    open-type="getUserInfo"
                    @getuserinfo="onGetUserInfo"
                    class="userinfo-avatar"
                    :style="'background-image: url(' + avatarUrl + ')'"
                    size="default"
                ></button>
            </block>
            <block v-else>
                <image @tap="bindViewTap" class="userinfo-block-avatar" :src="avatarUrl" mode="cover"></image>
            </block>
            <view class="userinfo-nickname-wrapper">
                <button class="userinfo-nickname" @tap="onGetOpenid">点击获取 openid</button>
            </view>
        </view>

        <!-- 上传图片 -->
        <view class="uploader">
            <view class="uploader-text" @tap="doUpload">
                <text>上传图片</text>
            </view>
            <view class="uploader-container" v-if="imgUrl">
                <image class="uploader-image" :src="imgUrl" mode="aspectFit" @tap="previewImg"></image>
            </view>
        </view>

        <!-- 操作数据库 -->
        <view class="uploader">
            <navigator url="../databaseGuide/databaseGuide" open-type="navigate" class="uploader-text">
                <text>前端操作数据库</text>
            </navigator>
        </view>

        <!-- 即时通信 -->
        <view class="uploader">
            <navigator url="../im/im" open-type="navigate" class="uploader-text">
                <text>即时通信 Demo</text>
            </navigator>
        </view>

        <!-- 新建云函数 -->
        <view class="uploader">
            <navigator url="../addFunction/addFunction" open-type="navigate" class="uploader-text">
                <text>快速新建云函数</text>
            </navigator>
        </view>

        <!-- 云调用 -->
        <view class="uploader">
            <navigator url="../openapi/openapi" open-type="navigate" class="uploader-text">
                <text>云调用</text>
            </navigator>
        </view>
    </view>
</template>

<script>
//index.js
const app = getApp();
export default {
    data() {
        return {
            avatarUrl: '/static/pages/index/user-unlogin.png',
            userInfo: {},
            hasUserInfo: false,
            logged: false,
            takeSession: false,
            requestResult: '',
            canIUseGetUserProfile: false,

            // 如需尝试获取用户信息可改为false
            canIUseOpenData: uni.canIUse('open-data.type.userAvatarUrl'),

            imgUrl: ''
        };
    },
    onLoad: function () {
        if (!wx.cloud) {
            uni.redirectTo({
                url: '../chooseLib/chooseLib'
            });
            return;
        }

        if (uni.getUserProfile) {
            this.setData({
                canIUseGetUserProfile: true
            });
        }
    },
    methods: {
        getUserProfile() {
            // 推荐使用wx.getUserProfile获取用户信息，开发者每次通过该接口获取用户个人信息均需用户确认，开发者妥善保管用户快速填写的头像昵称，避免重复弹窗
            uni.getUserProfile({
                desc: '展示用户信息',
                // 声明获取用户个人信息后的用途，后续会展示在弹窗中，请谨慎填写
                success: (res) => {
                    this.setData({
                        avatarUrl: res.userInfo.avatarUrl,
                        userInfo: res.userInfo,
                        hasUserInfo: true
                    });
                }
            });
        },

        onGetUserInfo: function (e) {
            if (!this.logged && e.detail.userInfo) {
                this.setData({
                    logged: true,
                    avatarUrl: e.detail.userInfo.avatarUrl,
                    userInfo: e.detail.userInfo,
                    hasUserInfo: true
                });
            }
        },

        onGetOpenid: function () {
            // 调用云函数
            wx.cloud.callFunction({
                name: 'login',
                data: {},
                success: (res) => {
                    console.log('[云函数] [login] user openid: ', res.result.openid);
                    app.globalData.openid = res.result.openid;
                    uni.navigateTo({
                        url: '../userConsole/userConsole'
                    });
                },
                fail: (err) => {
                    console.error('[云函数] [login] 调用失败', err);
                    uni.navigateTo({
                        url: '../deployFunctions/deployFunctions'
                    });
                }
            });
        },

        // 上传图片
        doUpload: function () {
            // 选择图片
            uni.chooseImage({
                count: 1,
                sizeType: ['compressed'],
                sourceType: ['album', 'camera'],
                success: function (res) {
                    uni.showLoading({
                        title: '上传中'
                    });
                    const filePath = res.tempFilePaths[0]; // 上传图片

                    const cloudPath = `my-image${filePath.match(/\.[^.]+?$/)[0]}`;
                    wx.cloud.uploadFile({
                        cloudPath,
                        filePath,
                        success: (res) => {
                            console.log('[上传文件] 成功：', res);
                            app.globalData.fileID = res.fileID;
                            app.globalData.cloudPath = cloudPath;
                            app.globalData.imagePath = filePath;
                            uni.navigateTo({
                                url: '../storageConsole/storageConsole'
                            });
                        },
                        fail: (e) => {
                            console.error('[上传文件] 失败：', e);
                            uni.showToast({
                                icon: 'none',
                                title: '上传失败'
                            });
                        },
                        complete: () => {
                            uni.hideLoading();
                        }
                    });
                },
                fail: (e) => {
                    console.error(e);
                }
            });
        },

        bindViewTap() {
            console.log('占位：函数 bindViewTap 未声明');
        },

        previewImg() {
            console.log('占位：函数 previewImg 未声明');
        }
    }
};
</script>
<style>
@import './index.css';
</style>

<template>
<view>
    <view class="photo_temple_box" style="
      background: #fff;
       border-radius: 16px 16px 0 0;">

        <!-- 背景图 -->
        <image id="bannerImage" :src="backGroundImageUrl" class="banner-img" />

        <!-- 头像区 -->
        <image class="userheadImage" id="userheadImage" :src="headImage" />

        <view class="context_box">
            <view id="context" class="sku-name">
                {{ contentName }}
            </view>
        </view>
        <view class="title-group">
            <view class="store-box">
                <image id="imgjp" class="store-url" :src="my" />

                <view id="qrCode" class="store-uscode" />
            </view>
        </view>

        <view class="store-copy">
            <text id="endText">
                — 长按识别海报 关注公众号 —
            </text>
        </view>
    </view>
    <canvas v-show="!completeCanvas" id="myQrcode" type="2d" class="canves" :style="canvasStyle" />
</view>
</template>

<script>
import Taro from '@tarojs/taro'
import QR from 'qrcode-base64'

import headImage from '../asset/touxiang.jpg'
import erweima from '../asset/erweima.jpg'
import bg from '../asset/bg.jpg'
import bg1 from '../asset/bg1.jpg'
import my from '../asset/my.jpg'

export default {
    /* 海报浏览器 */
    name: 'PhotoTemple',
    data() {
        const system = Taro.getSystemInfoSync()
        const {
            windowHeight,
            windowWidth,
        } = system
        return {
            canvasStyle: {
                width: windowWidth + 'px',
                opacity: 0,
                height: windowHeight + 'px',
            },
            windowWidth,
            my,
            headImage,
            scale: 0.5,
            startTop: 0,
            backGroundImageUrl: bg || bg1,
            /* cavnas区域的起始位置 */
            endTop: 0,
            /* cavnas区域的最终位置 */
            completeCanvas: false,
            /* 是否完成画布 */
            cavnsOffsetop: 20,
            erweima, 
            contentName: '把值得做的事坚持下去，再把坚持做的事努力做好,公众号 前端Sharing ,专注于分享前端技术，专研前端技术，编写前端工具，开源前端项目，希望能和大家，一起学习一起成长。最后祝大家新年快乐～'
            /* canvas 向上偏移量* */
        }
    },
    mounted() {
        Taro.nextTick(() => {
            const {
                windowHeight,
                /* 屏幕高度 */
                windowWidth,
                /* 屏幕宽度 */
                pixelRatio,
                /* 像素缩放比 */
            } = Taro.getSystemInfoSync()
            const query = Taro.createSelectorQuery()
            query.select('#myQrcode').fields({
                node: true,
                size: true
            }).exec(async res => {
                let {
                    node,
                } = res[0]
                if (!node) return
                const context = node.getContext('2d')
                Taro.showLoading({
                    title: '海报生成中'
                })
                const dpr = pixelRatio
                this.scale = dpr
                /* TODO: canvas 画布的宽高 和 元素的宽高 必须保持相同的长宽比列,否则会变形 */
                node.width = windowWidth * dpr
                node.height = windowHeight * dpr
                context.scale(dpr, dpr)
                context.fillStyle = '#fff'
                context.fillRect(0, 0, windowWidth, windowHeight)

                //1024 768

                /* TODO: banner图 */
                const bannerImage = await this.geDomPostion('#bannerImage')
                this.startTop = bannerImage.top - 30
                await this.drawImage(context, node, this.backGroundImageUrl, 0, 0, 1024, 768, 0, this.startTop, windowWidth, windowWidth)
                context.save()
                Taro.hideLoading()

                /*TODO: 绘制头像 ,及昵称 */
                const userheadImage = await this.geDomPostion('#userheadImage', true)
                /* 圆形图片 */
                let d = userheadImage.height / 2
                const cx = userheadImage.left + userheadImage.width / 2
                let cy = userheadImage.top + userheadImage.height / 2
                context.arc(cx, cy, d, 0, 2 * Math.PI)
                context.strokeStyle = '#FFFFFF'
                context.stroke()
                context.clip()
                await this.drawImage(context, node, this.headImage, userheadImage.left, userheadImage.top, userheadImage.width, userheadImage.height)
                context.restore()
                this.drawText(context, {
                    top: userheadImage.top + userheadImage.height + 40,
                    left: userheadImage.left - 70
                }, '我不是外星人「前端Sharing」', 18, 'normal 600 20px PingFangSC-Semibold', '#fff')

                /* TODO: 复制多行文本 */
                const rowsText = await this.geDomPostion('#context', true)
                this.drawRanksTexts(context, this.contentName, rowsText.top, rowsText.left, rowsText.width)

                /*TODO: 第四步：绘制二维码 */
                const qrCode = await this.geDomPostion('#qrCode', true)
                // await this.drawCode(context, node, qrCode.left - 20, qrCode.top - this.cavnsOffsetop)
                await this.drawImage(context, node, this.erweima, qrCode.left - 50, qrCode.top - 40, qrCode.width * 2, qrCode.height * 2)

                /* TODO: 第五步： 绘制长按复制 */
                const endText = await this.geDomPostion('#endText')
                this.endTop = endText.top + 10
                this.drawText(
                    context, {
                        ...endText,
                        top: endText.top - this.cavnsOffsetop + 10
                    },
                    '— 长按识别海报 关注公众号—',
                    13,
                    'normal 400 13px PingFangSC-Regular',
                    '#909399'
                )

                /*TODO: 第三步： 微信信息 */
                const imgjp = await this.geDomPostion('#imgjp', true)
                await this.drawImage(context, node, this.my, imgjp.left, imgjp.top - 30, imgjp.width, imgjp.height)

                this.drawText(
                    context, {
                        left: imgjp.left,
                        top: imgjp.top + imgjp.height
                    },
                    'vx: TH0000666',
                    13,
                    'normal 400 13px PingFangSC-Regular',
                    '#909399'
                )

                this.makePc(node)

            })
        })
    },
    methods: {

        /* 绘制直线 */
        drawline(ctx, from, to, lw, lc) {
            ctx.moveTo(...from)
            ctx.lineTo(...to)
            ctx.lineWidth = lw
            console.log(ctx)
            ctx.strokeStyle = lc
            ctx.stroke()
        },
        /** 画多行文本
         * @param ctx          canvas 上下文
         * @param str          多行文本
         * @param initHeight   容器初始 top值
         * @param initWidth    容器初始 left值
         * @param canvasWidth  容器宽度
         */
        drawRanksTexts(ctx, str, initHeight, initWidth, canvasWidth) {
            let lineWidth = 0;
            let lastSubStrIndex = 0;
            /* 设置文字样式 */
            ctx.fillStyle = "#303133"
            ctx.font = 'normal 400 15px  PingFangSC-Regular'
            for (let i = 0; i < str.length; i++) {
                lineWidth += ctx.measureText(str[i]).width
                if (lineWidth > canvasWidth) {
                    ctx.fillText(str.substring(lastSubStrIndex, i), initWidth, initHeight)
                    initHeight += 20
                    lineWidth = 0
                    lastSubStrIndex = i
                }
                if (i == str.length - 1) {
                    ctx.fillText(str.substring(lastSubStrIndex, i + 1), initWidth, initHeight)
                }
            }

        },
        /* 画文本 */
        drawText(ctx, result, text, fz, ff, fc) {
            ctx.fillStyle = fc
            ctx.fontSize = fz
            ctx.font = ff
            ctx.fillText(text, result.left, result.top)
            ctx.save()
        },
        /* 获取元素位置 */
        geDomPostion(dom, isAll) {
            return new Promise((resolve) => {
                Taro.createSelectorQuery().select(dom).boundingClientRect(rect => {
                    const {
                        top,
                        left
                    } = rect
                    resolve(isAll ? rect : {
                        top,
                        left
                    })
                }).exec()
            })
        },
        /* 获取网路图片信息 */
        getImageInfo(src) {
            return new Promise((resolve) => {
                wx.getImageInfo({
                    src,
                    success: (res) => {
                        resolve(res)
                    },
                })
            })
        },
        /* 图片 */
        drawImage(context, node, url, ...arg) {
            return new Promise((resolve) => {
                const image = node.createImage()
                image.src = url
                image.onload = () => {

                    context.drawImage(image, ...arg)
                    resolve()
                }
            })
        },
        /* 生成二维码 */
        drawCode(ctx, node, x, y) {
            return new Promise((resolve) => {
                const codeImageWidth = 150    /* 绘制二维码宽度           */
                const canvasImageWidth = 85   /* 二维码绘制到canvas的宽度  */
                const left = x - 15           /* left 值                */
                const top = y - 22            /* top 值                 */
                const logoWidth = 15          /* 二维码中logo宽度  */

                const base64 = QR.drawImg('https://juejin.cn/user/2418581313687390', {
                    typeNumber: 4,
                    errorCorrectLevel: 'L',
                    size: codeImageWidth
                })
                /* 创建读写流 */
                const fs = Taro.getFileSystemManager()
                const times = new Date().getTime()
                const codeimg = Taro.env.USER_DATA_PATH + '/' + times + '.png'

                /* 将base64图片写入 */
                fs.writeFile({
                    filePath: codeimg,
                    data: base64.slice(22),
                    encoding: 'base64',
                    success: async () => {
                        const offset = (canvasImageWidth - logoWidth) / 2 /* 偏移量 */
                        await this.drawImage(ctx, node, codeimg, 0, 0, codeImageWidth, codeImageWidth, left, top, canvasImageWidth, canvasImageWidth)
                        await this.drawImage(ctx, node, this.logoUrl, left + offset, top + offset, logoWidth, logoWidth) 
                        resolve()
                    }
                })
            })

        },
        /* 生成海报 */
        makePc(node) {
            const {
                startTop,
                endTop,
                windowWidth
            } = this
            Taro.canvasToTempFilePath({
                x: 0,
                y: startTop,
                width: windowWidth,
                height: endTop - startTop,
                destWidth: windowWidth * 3,
                destHeight: (endTop - startTop) * 3,
                canvas: node,
                success: function (res) {
                    Taro.hideLoading()
                    Taro.previewImage({
                        urls: [res.tempFilePath]
                    })

                }
            })
        }
    }
}
</script>

<style lang="scss">
.myQrcode {
    position: fixed;
    height: 100%;
    width: 100%;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
}

.photo_temple_box {
    opacity: 1;
    position: absolute;
    width: 100%;
    top: 50%;
    transform: translateY(-50%);

    .userheadImage {
        position: fixed;
        height: 200px;
        width: 200px;
        left: 50%;
        top: 30%;
        transform: translate(-50%, -50%);
    }

    #text1 {
        font-size: 40px;
    }

    #text2 {
        font-size: 54px;
    }

    #text3 {
        font-size: 36px;
    }

    img {
        width: 400px;
        height: 400pX;
    }

    .context_box {
        padding-left: 30px;
        padding-right: 30px;
        margin-top: 32px;
    }

    .context {
        font-family: PingFangSC-Regular;
        font-size: 30px;
        color: #303133;
        letter-spacing: 0;
        line-height: 45px;

    }

    .canvesbox {
        position: absolute;
        height: 100px;
        width: 100px;
        // overflow: hidden;
        z-index: 10003;

        .canves {
            transform: translate(580px, -180px);
        }
    }

    #capture {
        width: 80%;
        margin: auto;
        background: #fff;
        margin-top: 10%;
        /* pointer-events: none; */
    }

    .title-group {
        margin: 0 24px;
        text-align: left;
        background: #fff;
    }

    .category-id {
        font-family: PingFangSC-Semibold;
        letter-spacing: 0;
        color: #F12E40;
    }

    .font-value {
        display: flex;
        -webkit-align-items: baseline;
        -ms-flex-align: baseline;
        align-items: baseline;
        padding-left: 15px;
        margin: 10px 0;
    }

    .category-id-name {
        font-family: PingFangSC-Semibold;
        font-size: 26px;
        letter-spacing: 0;
        padding-right: 12px;
        display: inline-block;
    }

    .category-id-value {
        font-family: PingFangSC-Semibold;
        font-variant: normal !important;
        font-size: 40px;
        letter-spacing: 1px;
        display: inline-block;
        font-weight: 600;
        margin-left: -5px;
        transform: translateY(1.5px);
    }

    .lineThrough {
        display: inline-block;
        transform: translateX(-60px);
    }

    .upc-code {
        font-family: PingFangSC-Semibold;
        font-size: 30px;
        color: #C0C4CC;
        letter-spacing: 0;
        display: inline-block;
        // text-decoration: line-through;
        margin-left: 20px;
        transform: translateY(1.5px);
        position: relative;
    }

    .category-id .brand-name {
        font-family: PingFangSC-Semibold;
        font-size: 31px;
        color: #FFFFFF;
        letter-spacing: 0;
        background-image: linear-gradient(-45deg, #FF842E 0%, #FF3934 25%, #FB23C9 100%);
        width: 178px;
        height: 46px;
        display: inline-block;
        text-align: right;
        width: 40%;
    }

    .category-id .brand-good {
        font-family: PingFangSC-Regular;
        font-size: 26px;
        color: #909399;
        letter-spacing: 0;
        text-align: right;
        float: right;
        position: relative;
        margin-left: auto;
        /* top: 22px; */
    }

    .sku-name {
        font-family: PingFangSC-Semibold;
        font-size: 30px;
        color: #303133;
        letter-spacing: 0.5px;
        line-height: 45px;
        /* min-height: 90px; */
        word-break: break-all;
        text-align: justify;
    }

    .store-box {
        margin-top: 38px;
        display: flex;
        align-items: center;
    }

    .store-url {
        width: 150px;
        height: 150px;
        /* float: left; */
    }

    .banner-img {
        height: 700px;

        width: 100%;
        border-radius: 16px 16px 0 0;
    }

    .store-uscode {
        width: 100px;
        height: 100px;
        /* float: right; */
        margin-left: auto;

        /* margin-right: 21px; */
        .canves {
            width: 100%;
            height: 100%;
        }
    }

    .store-wrap {
        display: inline-block;
        text-align: left;
    }

    .store-nick-name {
        font-family: JDLANGZHENGTI--GB1-0;
        font-size: 32px;
        color: #303133;
        letter-spacing: 0;
        text-align: left;
        line-height: 36px;
        margin-top: 10px;
        font-weight: 600;
    }

    .store-content {
        font-family: PingFangSC-Regular;
        font-size: 24px;
        color: #909399;
        letter-spacing: 0;
        text-align: left;
        line-height: 36px;
        margin-top: 10px;
    }

    .store-copy {
        /* margin-top:30px;
        margin-bottom:20px; */
        font-family: PingFangSC-Regular;
        font-size: 26px;
        color: #909399;
        letter-spacing: 0;
        text-align: center;
        height: 76px;
        line-height: 76px;
        background: rgb(255, 255, 255);
        border-radius: 0px 0px 16px 16px;
        margin-top: -1px;
    }
}
</style>

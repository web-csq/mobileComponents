<template>
    <div idm-ctrl="idm_module" :id="moduleObject.id" :idm-ctrl-id="moduleObject.id" class="idm-navbar">
        <!--
           组件内部容器
           增加class="drag_container" 必选
           idm-ctrl-id：组件的id，这个必须不能为空
           idm-container-index  组件的内部容器索引，不重复唯一且不变，必选
         -->
        <van-nav-bar
            :placeholder="propData.isPlaceholder"
            :z-index="propData.zIndex"
            :safe-area-inset-top="propData.isSafeAreaInsetTop"
        >
            <template #left>
                <div class="idm-navbar-left-container" @click="handleLeftClick">
                    <div class="align-center" v-if="propData.isShowLeftIcon">
                        <svg
                            v-if="propData.leftIcon && propData.leftIcon.length"
                            class="idm-navbar-left-icon"
                            aria-hidden="true"
                        >
                            <use :xlink:href="`#${propData.leftIcon[0]}`"></use>
                        </svg>
                        <div v-else class="idm-navbar-left-icon align-center">
                            <svg-icon icon-class="arrow-right"></svg-icon>
                        </div>
                    </div>
                    <div v-if="propData.leftText" class="idm-navbar-left-text">
                        {{ propData.leftText }}
                    </div>
                </div>
            </template>
            <template #title>
                <div class="idm-navbar-title" @click="handleTitleClick">
                    {{ propData.titleText }}
                </div>
            </template>
            <template #right>
                <div class="idm-navbar-right-container" @click="handleRightClick">
                    <div v-if="propData.rightText" class="idm-navbar-right-text">{{ propData.rightText }}</div>
                    <div class="align-center" v-if="propData.isShowRightIcon">
                        <svg
                            v-if="propData.rightIcon && propData.rightIcon.length"
                            class="idm-navbar-right-icon"
                            aria-hidden="true"
                        >
                            <use :xlink:href="`#${propData.rightIcon[0]}`"></use>
                        </svg>
                        <div v-else class="idm-navbar-right-icon align-center">
                            <svg-icon icon-class="search"></svg-icon>
                        </div>
                    </div>
                </div>
            </template>
        </van-nav-bar>
    </div>
</template>
<script>
import NavBar from 'vant/lib/nav-bar'
export default {
    name: 'INavBar',
    components: {
        [NavBar.name]: NavBar
    },
    data() {
        return {
            moduleObject: {},
            propData: this.$root.propData.compositeAttr || {}
        }
    },
    created() {
        this.moduleObject = this.$root.moduleObject
        this.convertAttrToStyleObject()
        this.convertThemeListAttrToStyleObject()
    },
    methods: {
        /**
         * 通用的url参数对象
         * 所有地址的url参数转换
         */
        commonParam() {
            let urlObject = IDM.url.queryObject()
            var params = {
                pageId:
                    window.IDM.broadcast && window.IDM.broadcast.pageModule ? window.IDM.broadcast.pageModule.id : '',
                urlData: JSON.stringify(urlObject)
            }
            return params
        },
        /**
         * 通用自定义函数
         */
        eventFunHandle(name) {
            var customHandle = this.propData[name]
            customHandle &&
                customHandle.forEach((item) => {
                    window[item.name] &&
                        window[item.name].call(this, {
                            ...this.commonParam(),
                            customParam: item.param,
                            _this: this
                        })
                })
        },
        handleLeftClick() {
            this.eventFunHandle('leftCustomFunction')
        },
        handleTitleClick() {
            this.eventFunHandle('titleCustomFunction')
        },
        handleRightClick() {
            this.eventFunHandle('rightCustomFunction')
        },
        propDataWatchHandle(propData) {
            this.propData = propData.compositeAttr || {}
            this.convertAttrToStyleObject()
            this.convertThemeListAttrToStyleObject()
        },
        convertAttrToStyleObject() {
            const styleObject = {},
                heightObj = {},
                titleFontObj = {},
                leftIconObj = {},
                rightIconObj = {},
                leftFontObj = {},
                rightFontObj = {}
            if (this.propData.bgSize && this.propData.bgSize == 'custom') {
                styleObject['background-size'] =
                    (this.propData.bgSizeWidth
                        ? this.propData.bgSizeWidth.inputVal + this.propData.bgSizeWidth.selectVal
                        : 'auto') +
                    ' ' +
                    (this.propData.bgSizeHeight
                        ? this.propData.bgSizeHeight.inputVal + this.propData.bgSizeHeight.selectVal
                        : 'auto')
            } else if (this.propData.bgSize) {
                styleObject['background-size'] = this.propData.bgSize
            }
            if (this.propData.positionX && this.propData.positionX.inputVal) {
                styleObject['background-position-x'] =
                    this.propData.positionX.inputVal + this.propData.positionX.selectVal
            }
            if (this.propData.positionY && this.propData.positionY.inputVal) {
                styleObject['background-position-y'] =
                    this.propData.positionY.inputVal + this.propData.positionY.selectVal
            }
            for (const key in this.propData) {
                if (this.propData.hasOwnProperty.call(this.propData, key)) {
                    const element = this.propData[key]
                    if (!element && element !== false && element != 0) {
                        continue
                    }
                    switch (key) {
                        case 'leftIconColor':
                            leftIconObj['fill'] = element.hex8 + ' !important'
                            leftIconObj['color'] = element.hex8 + ' !important'
                            break
                        case 'leftIconSize':
                            leftIconObj['width'] = element + 'px'
                            leftIconObj['height'] = element + 'px'
                            leftIconObj['font-size'] = element + 'px'
                            break
                        case 'rightIconColor':
                            rightIconObj['fill'] = element.hex8 + ' !important'
                            rightIconObj['color'] = element.hex8 + ' !important'
                            break
                        case 'rightIconSize':
                            rightIconObj['width'] = element + 'px'
                            rightIconObj['height'] = element + 'px'
                            rightIconObj['font-size'] = element + 'px'
                            break
                        case 'width':
                            styleObject[key] = element
                            break
                        case 'height':
                            styleObject[key] = element
                            heightObj[key] = element
                            break
                        case 'bgColor':
                            if (element && element.hex8) {
                                styleObject['background-color'] = element.hex8
                            }
                            break
                        case 'box':
                            if (element.marginTopVal) {
                                styleObject['margin-top'] = `${element.marginTopVal}`
                            }
                            if (element.marginRightVal) {
                                styleObject['margin-right'] = `${element.marginRightVal}`
                            }
                            if (element.marginBottomVal) {
                                styleObject['margin-bottom'] = `${element.marginBottomVal}`
                            }
                            if (element.marginLeftVal) {
                                styleObject['margin-left'] = `${element.marginLeftVal}`
                            }
                            if (element.paddingTopVal) {
                                styleObject['padding-top'] = `${element.paddingTopVal}`
                            }
                            if (element.paddingRightVal) {
                                styleObject['padding-right'] = `${element.paddingRightVal}`
                            }
                            if (element.paddingBottomVal) {
                                styleObject['padding-bottom'] = `${element.paddingBottomVal}`
                            }
                            if (element.paddingLeftVal) {
                                styleObject['padding-left'] = `${element.paddingLeftVal}`
                            }
                            break
                        case 'leftIconBox':
                            if (element.marginTopVal) {
                                leftIconObj['margin-top'] = `${element.marginTopVal}`
                            }
                            if (element.marginRightVal) {
                                leftIconObj['margin-right'] = `${element.marginRightVal}`
                            }
                            if (element.marginBottomVal) {
                                leftIconObj['margin-bottom'] = `${element.marginBottomVal}`
                            }
                            if (element.marginLeftVal) {
                                leftIconObj['margin-left'] = `${element.marginLeftVal}`
                            }
                            if (element.paddingTopVal) {
                                leftIconObj['padding-top'] = `${element.paddingTopVal}`
                            }
                            if (element.paddingRightVal) {
                                leftIconObj['padding-right'] = `${element.paddingRightVal}`
                            }
                            if (element.paddingBottomVal) {
                                leftIconObj['padding-bottom'] = `${element.paddingBottomVal}`
                            }
                            if (element.paddingLeftVal) {
                                leftIconObj['padding-left'] = `${element.paddingLeftVal}`
                            }
                            break
                        case 'rightIconBox':
                            if (element.marginTopVal) {
                                rightIconObj['margin-top'] = `${element.marginTopVal}`
                            }
                            if (element.marginRightVal) {
                                rightIconObj['margin-right'] = `${element.marginRightVal}`
                            }
                            if (element.marginBottomVal) {
                                rightIconObj['margin-bottom'] = `${element.marginBottomVal}`
                            }
                            if (element.marginLeftVal) {
                                rightIconObj['margin-left'] = `${element.marginLeftVal}`
                            }
                            if (element.paddingTopVal) {
                                rightIconObj['padding-top'] = `${element.paddingTopVal}`
                            }
                            if (element.paddingRightVal) {
                                rightIconObj['padding-right'] = `${element.paddingRightVal}`
                            }
                            if (element.paddingBottomVal) {
                                rightIconObj['padding-bottom'] = `${element.paddingBottomVal}`
                            }
                            if (element.paddingLeftVal) {
                                rightIconObj['padding-left'] = `${element.paddingLeftVal}`
                            }
                            break
                        case 'leftTextBox':
                            if (element.marginTopVal) {
                                leftFontObj['margin-top'] = `${element.marginTopVal}`
                            }
                            if (element.marginRightVal) {
                                leftFontObj['margin-right'] = `${element.marginRightVal}`
                            }
                            if (element.marginBottomVal) {
                                leftFontObj['margin-bottom'] = `${element.marginBottomVal}`
                            }
                            if (element.marginLeftVal) {
                                leftFontObj['margin-left'] = `${element.marginLeftVal}`
                            }
                            if (element.paddingTopVal) {
                                leftFontObj['padding-top'] = `${element.paddingTopVal}`
                            }
                            if (element.paddingRightVal) {
                                leftFontObj['padding-right'] = `${element.paddingRightVal}`
                            }
                            if (element.paddingBottomVal) {
                                leftFontObj['padding-bottom'] = `${element.paddingBottomVal}`
                            }
                            if (element.paddingLeftVal) {
                                leftFontObj['padding-left'] = `${element.paddingLeftVal}`
                            }
                            break
                        case 'rightTextBox':
                            if (element.marginTopVal) {
                                rightFontObj['margin-top'] = `${element.marginTopVal}`
                            }
                            if (element.marginRightVal) {
                                rightFontObj['margin-right'] = `${element.marginRightVal}`
                            }
                            if (element.marginBottomVal) {
                                rightFontObj['margin-bottom'] = `${element.marginBottomVal}`
                            }
                            if (element.marginLeftVal) {
                                rightFontObj['margin-left'] = `${element.marginLeftVal}`
                            }
                            if (element.paddingTopVal) {
                                rightFontObj['padding-top'] = `${element.paddingTopVal}`
                            }
                            if (element.paddingRightVal) {
                                rightFontObj['padding-right'] = `${element.paddingRightVal}`
                            }
                            if (element.paddingBottomVal) {
                                rightFontObj['padding-bottom'] = `${element.paddingBottomVal}`
                            }
                            if (element.paddingLeftVal) {
                                rightFontObj['padding-left'] = `${element.paddingLeftVal}`
                            }
                            break
                        case 'bgImgUrl':
                            styleObject['background-image'] = `url(${IDM.url.getWebPath(element)})`
                            break
                        case 'positionX':
                            //背景横向偏移

                            break
                        case 'positionY':
                            //背景纵向偏移

                            break
                        case 'bgRepeat':
                            //平铺模式
                            styleObject['background-repeat'] = element
                            break
                        case 'bgAttachment':
                            //背景模式
                            styleObject['background-attachment'] = element
                            break
                        case 'border':
                            if (element.border.top.width > 0) {
                                styleObject['border-top-width'] =
                                    element.border.top.width + element.border.top.widthUnit
                                styleObject['border-top-style'] = element.border.top.style
                                if (element.border.top.colors.hex8) {
                                    styleObject['border-top-color'] = element.border.top.colors.hex8
                                }
                            }
                            if (element.border.right.width > 0) {
                                styleObject['border-right-width'] =
                                    element.border.right.width + element.border.right.widthUnit
                                styleObject['border-right-style'] = element.border.right.style
                                if (element.border.right.colors.hex8) {
                                    styleObject['border-right-color'] = element.border.right.colors.hex8
                                }
                            }
                            if (element.border.bottom.width > 0) {
                                styleObject['border-bottom-width'] =
                                    element.border.bottom.width + element.border.bottom.widthUnit
                                styleObject['border-bottom-style'] = element.border.bottom.style
                                if (element.border.bottom.colors.hex8) {
                                    styleObject['border-bottom-color'] = element.border.bottom.colors.hex8
                                }
                            }
                            if (element.border.left.width > 0) {
                                styleObject['border-left-width'] =
                                    element.border.left.width + element.border.left.widthUnit
                                styleObject['border-left-style'] = element.border.left.style
                                if (element.border.left.colors.hex8) {
                                    styleObject['border-left-color'] = element.border.left.colors.hex8
                                }
                            }

                            styleObject['border-top-left-radius'] =
                                element.radius.leftTop.radius + element.radius.leftTop.radiusUnit
                            styleObject['border-top-right-radius'] =
                                element.radius.rightTop.radius + element.radius.rightTop.radiusUnit
                            styleObject['border-bottom-left-radius'] =
                                element.radius.leftBottom.radius + element.radius.leftBottom.radiusUnit
                            styleObject['border-bottom-right-radius'] =
                                element.radius.rightBottom.radius + element.radius.rightBottom.radiusUnit
                            break
                        case 'titleFont':
                            titleFontObj['font-family'] = element.fontFamily
                            if (element.fontColors.hex8) {
                                titleFontObj['color'] = element.fontColors.hex8
                            }
                            titleFontObj['font-weight'] = element.fontWeight && element.fontWeight.split(' ')[0]
                            titleFontObj['font-style'] = element.fontStyle
                            titleFontObj['font-size'] = element.fontSize + element.fontSizeUnit
                            titleFontObj['line-height'] =
                                element.fontLineHeight +
                                (element.fontLineHeightUnit == '-' ? '' : element.fontLineHeightUnit)
                            titleFontObj['text-align'] = element.fontTextAlign
                            titleFontObj['text-decoration'] = element.fontDecoration
                            break
                        case 'leftFont':
                            leftFontObj['font-family'] = element.fontFamily
                            if (element.fontColors.hex8) {
                                leftFontObj['color'] = element.fontColors.hex8 + ' !important'
                            }
                            leftFontObj['font-weight'] = element.fontWeight && element.fontWeight.split(' ')[0]
                            leftFontObj['font-style'] = element.fontStyle
                            leftFontObj['font-size'] = element.fontSize + element.fontSizeUnit
                            leftFontObj['line-height'] =
                                element.fontLineHeight +
                                (element.fontLineHeightUnit == '-' ? '' : element.fontLineHeightUnit)
                            leftFontObj['text-align'] = element.fontTextAlign
                            leftFontObj['text-decoration'] = element.fontDecoration
                            break
                        case 'rightFont':
                            rightFontObj['font-family'] = element.fontFamily
                            if (element.fontColors.hex8) {
                                rightFontObj['color'] = element.fontColors.hex8 + ' !important'
                            }
                            rightFontObj['font-weight'] = element.fontWeight && element.fontWeight.split(' ')[0]
                            rightFontObj['font-style'] = element.fontStyle
                            rightFontObj['font-size'] = element.fontSize + element.fontSizeUnit
                            rightFontObj['line-height'] =
                                element.fontLineHeight +
                                (element.fontLineHeightUnit == '-' ? '' : element.fontLineHeightUnit)
                            rightFontObj['text-align'] = element.fontTextAlign
                            rightFontObj['text-decoration'] = element.fontDecoration
                            break
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id, styleObject)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-nav-bar__content', heightObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-nav-bar__title', titleFontObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-navbar-left-icon', leftIconObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-navbar-right-icon', rightIconObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-navbar-left-text', leftFontObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-navbar-right-text', rightFontObj)
            this.initData()
        },
        /**
         * 批量生成css类名
         * @param {前缀选择器} selectorPrefix
         * @param {子级选择器} subSelectorArray
         * @returns css 选择器字符串
         */
        generateClassName(selectorPrefix, subSelectorArray) {
            const selectorStr = subSelectorArray.map((el) => selectorPrefix + el).join(',')
            if (selectorStr.startsWith('#')) return selectorStr.substr(1)
            return selectorStr
        },
        /** 通用主题颜色 */
        convertThemeListAttrToStyleObject() {
            var themeList = this.propData.themeList
            if (!themeList) {
                return
            }
            const themeNamePrefix =
                IDM.setting && IDM.setting.applications && IDM.setting.applications.themeNamePrefix
                    ? IDM.setting.applications.themeNamePrefix
                    : 'idm-theme-'
            for (var i = 0; i < themeList.length; i++) {
                var item = themeList[i]
                const mainColor = item.mainColor ? IDM.hex8ToRgbaString(item.mainColor.hex8) : ''
                const mainColorObj = {
                    color: mainColor,
                    fill: mainColor
                }
                IDM.setStyleToPageHead(
                    this.generateClassName('.' + themeNamePrefix + item.key, [
                        ` #${this.moduleObject.id} .idm-navbar-left-icon`,
                        ` #${this.moduleObject.id} .idm-navbar-right-icon`,
                        ` #${this.moduleObject.id} .idm-navbar-left-text`,
                        ` #${this.moduleObject.id} .idm-navbar-right-text`,
                    ]),
                    mainColorObj
                )
            }
        },
        reload() {
            //请求数据源
            this.initData()
        },
        initData() {
            let that = this
            //所有地址的url参数转换
            var params = that.commonParam()
            switch (this.propData.dataSourceType) {
                case 'customInterface':
                    this.propData.customInterfaceUrl &&
                        window.IDM.http
                            .get(this.propData.customInterfaceUrl, params)
                            .then((res) => {
                                //res.data
                                that.$set(
                                    that.propData,
                                    'fontContent',
                                    that.getExpressData('resultData', that.propData.dataFiled, res.data)
                                )
                                // that.propData.fontContent = ;
                            })
                            .catch(function (error) {})
                    break
                case 'pageCommonInterface':
                    //使用通用接口直接跳过，在setContextValue执行
                    break
                case 'customFunction':
                    if (this.propData.customFunction && this.propData.customFunction.length > 0) {
                        var resValue = ''
                        try {
                            resValue =
                                window[this.propData.customFunction[0].name] &&
                                window[this.propData.customFunction[0].name].call(this, {
                                    ...params,
                                    ...this.propData.customFunction[0].param,
                                    moduleObject: this.moduleObject
                                })
                        } catch (error) {}
                        that.propData.fontContent = resValue
                    }
                    break
            }
        },
        receiveBroadcastMessage(object) {
            console.log('组件收到消息', object)
            if (object.type && object.type == 'linkageShowModule') {
                this.showThisModuleHandle()
            } else if (object.type && object.type == 'linkageHideModule') {
                this.hideThisModuleHandle()
            }
        },
        setContextValue(object) {
            console.log('统一接口设置的值', object)
            if (object.type != 'pageCommonInterface') {
                return
            }
            //这里使用的是子表，所以要循环匹配所有子表的属性然后再去设置修改默认值
            if (object.key == this.propData.dataName) {
                // this.propData.fontContent = this.getExpressData(this.propData.dataName,this.propData.dataFiled,object.data);
                this.$set(
                    this.propData,
                    'fontContent',
                    this.getExpressData(this.propData.dataName, this.propData.dataFiled, object.data)
                )
            }
        },
        sendBroadcastMessage(object) {
            window.IDM.broadcast && window.IDM.broadcast.send(object)
        }
    }
}
</script>

<style lang="scss" scoped>
@import '~vant/lib/nav-bar/index.css';
$iconSize: 18px;
.align-center {
    display: flex;
    align-items: center;
}
.icon-size {
    width: $iconSize;
    height: $iconSize;
    font-size: $iconSize;
}
.idm-navbar {
    & ::v-deep() .van-nav-bar {
        background: transparent;
    }
    & ::v-deep() .van-nav-bar__title {
        min-width: 50%;
    }
    .idm-navbar-left-container {
        @extend .align-center;
        .idm-navbar-left-icon {
            @extend .icon-size;
        }
    }
    .idm-navbar-right-container {
        @extend .align-center;
        .idm-navbar-right-icon {
            @extend .icon-size;
        }
    }
}
</style>

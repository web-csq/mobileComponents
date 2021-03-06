<template>
    <div idm-ctrl="idm_module" :id="moduleObject.id" :idm-ctrl-id="moduleObject.id" class="idm-navbar">
        <van-nav-bar
            :placeholder="propData.isPlaceholder"
            :z-index="propData.zIndex"
            :safe-area-inset-top="propData.isSafeAreaInsetTop"
        >
            <!-- 左侧插槽 -->
            <template #left>
                <div class="idm-navbar-left-container" @click="handleLeftClick">
                    <!-- 左侧图标 -->
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
                    <!-- 左侧文本 -->
                    <div v-if="propData.leftText" class="idm-navbar-left-text">
                        {{ propData.leftText }}
                    </div>
                </div>
            </template>
            <!-- 标题插槽 -->
            <template #title>
                <div class="idm-navbar-title" @click="handleTitleClick">
                    {{ propData.titleText }}
                </div>
            </template>
            <!-- 右侧插槽 -->
            <template #right>
                <div class="idm-navbar-right-container" @click="handleRightClick">
                    <!-- 右侧文本 -->
                    <div v-if="propData.rightText" class="idm-navbar-right-text">{{ propData.rightText }}</div>
                    <!-- 右侧图标 -->
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
        // 通用的url参数对象
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
         * @param {函数名称} name
         */
        eventFunHandle(name) {
            if (this.moduleObject.env === 'develop') return
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
        // 左侧点击事件
        handleLeftClick() {
            if (this.propData.leftCustomFunction && this.propData.leftCustomFunction.length > 0) {
                this.eventFunHandle('leftCustomFunction')
                return
            }
            // 默认返回上一级子页面
            const currentId = this.moduleObject.routerId
            if (currentId) {
                IDM.router.back(currentId)
            }
        },
        // 标题点击事件
        handleTitleClick() {
            if (this.propData.titleCustomFunction && this.propData.titleCustomFunction.length > 0) {
                this.eventFunHandle('titleCustomFunction')
                return
            }
            window.scrollTo({
                top: 0,
                behavior: 'smooth'
            })
        },
        // 右侧点击事件
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
                        case 'boxShadow':
                            styleObject['box-shadow'] = element
                            break
                        case 'leftIconColor':
                            if (element.hex8) {
                                leftIconObj['fill'] = IDM.hex8ToRgbaString(element.hex8) + ' !important'
                                leftIconObj['color'] = IDM.hex8ToRgbaString(element.hex8) + ' !important'
                            }
                            break
                        case 'leftIconSize':
                            leftIconObj['width'] = element + 'px'
                            leftIconObj['height'] = element + 'px'
                            leftIconObj['font-size'] = element + 'px'
                            break
                        case 'rightIconColor':
                            if (element.hex8) {
                                rightIconObj['fill'] = IDM.hex8ToRgbaString(element.hex8) + ' !important'
                                rightIconObj['color'] = IDM.hex8ToRgbaString(element.hex8) + ' !important'
                            }
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
                                styleObject['background-color'] = IDM.hex8ToRgbaString(element.hex8)
                            }
                            break
                        case 'box':
                            if (element.marginTopVal) {
                                styleObject['margin-top'] = element.marginTopVal
                            }
                            if (element.marginRightVal) {
                                styleObject['margin-right'] = element.marginRightVal
                            }
                            if (element.marginBottomVal) {
                                styleObject['margin-bottom'] = element.marginBottomVal
                            }
                            if (element.marginLeftVal) {
                                styleObject['margin-left'] = element.marginLeftVal
                            }
                            if (element.paddingTopVal) {
                                styleObject['padding-top'] = element.paddingTopVal
                            }
                            if (element.paddingRightVal) {
                                styleObject['padding-right'] = element.paddingRightVal
                            }
                            if (element.paddingBottomVal) {
                                styleObject['padding-bottom'] = element.paddingBottomVal
                            }
                            if (element.paddingLeftVal) {
                                styleObject['padding-left'] = element.paddingLeftVal
                            }
                            break
                        case 'leftIconBox':
                            if (element.marginTopVal) {
                                leftIconObj['margin-top'] = element.marginTopVal
                            }
                            if (element.marginRightVal) {
                                leftIconObj['margin-right'] = element.marginRightVal
                            }
                            if (element.marginBottomVal) {
                                leftIconObj['margin-bottom'] = element.marginBottomVal
                            }
                            if (element.marginLeftVal) {
                                leftIconObj['margin-left'] = element.marginLeftVal
                            }
                            if (element.paddingTopVal) {
                                leftIconObj['padding-top'] = element.paddingTopVal
                            }
                            if (element.paddingRightVal) {
                                leftIconObj['padding-right'] = element.paddingRightVal
                            }
                            if (element.paddingBottomVal) {
                                leftIconObj['padding-bottom'] = element.paddingBottomVal
                            }
                            if (element.paddingLeftVal) {
                                leftIconObj['padding-left'] = element.paddingLeftVal
                            }
                            break
                        case 'rightIconBox':
                            if (element.marginTopVal) {
                                rightIconObj['margin-top'] = element.marginTopVal
                            }
                            if (element.marginRightVal) {
                                rightIconObj['margin-right'] = element.marginRightVal
                            }
                            if (element.marginBottomVal) {
                                rightIconObj['margin-bottom'] = element.marginBottomVal
                            }
                            if (element.marginLeftVal) {
                                rightIconObj['margin-left'] = element.marginLeftVal
                            }
                            if (element.paddingTopVal) {
                                rightIconObj['padding-top'] = element.paddingTopVal
                            }
                            if (element.paddingRightVal) {
                                rightIconObj['padding-right'] = element.paddingRightVal
                            }
                            if (element.paddingBottomVal) {
                                rightIconObj['padding-bottom'] = element.paddingBottomVal
                            }
                            if (element.paddingLeftVal) {
                                rightIconObj['padding-left'] = element.paddingLeftVal
                            }
                            break
                        case 'leftTextBox':
                            if (element.marginTopVal) {
                                leftFontObj['margin-top'] = element.marginTopVal
                            }
                            if (element.marginRightVal) {
                                leftFontObj['margin-right'] = element.marginRightVal
                            }
                            if (element.marginBottomVal) {
                                leftFontObj['margin-bottom'] = element.marginBottomVal
                            }
                            if (element.marginLeftVal) {
                                leftFontObj['margin-left'] = element.marginLeftVal
                            }
                            if (element.paddingTopVal) {
                                leftFontObj['padding-top'] = element.paddingTopVal
                            }
                            if (element.paddingRightVal) {
                                leftFontObj['padding-right'] = element.paddingRightVal
                            }
                            if (element.paddingBottomVal) {
                                leftFontObj['padding-bottom'] = element.paddingBottomVal
                            }
                            if (element.paddingLeftVal) {
                                leftFontObj['padding-left'] = element.paddingLeftVal
                            }
                            break
                        case 'rightTextBox':
                            if (element.marginTopVal) {
                                rightFontObj['margin-top'] = element.marginTopVal
                            }
                            if (element.marginRightVal) {
                                rightFontObj['margin-right'] = element.marginRightVal
                            }
                            if (element.marginBottomVal) {
                                rightFontObj['margin-bottom'] = element.marginBottomVal
                            }
                            if (element.marginLeftVal) {
                                rightFontObj['margin-left'] = element.marginLeftVal
                            }
                            if (element.paddingTopVal) {
                                rightFontObj['padding-top'] = element.paddingTopVal
                            }
                            if (element.paddingRightVal) {
                                rightFontObj['padding-right'] = element.paddingRightVal
                            }
                            if (element.paddingBottomVal) {
                                rightFontObj['padding-bottom'] = element.paddingBottomVal
                            }
                            if (element.paddingLeftVal) {
                                rightFontObj['padding-left'] = element.paddingLeftVal
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
                                    styleObject['border-top-color'] = IDM.hex8ToRgbaString(
                                        element.border.top.colors.hex8
                                    )
                                }
                            }
                            if (element.border.right.width > 0) {
                                styleObject['border-right-width'] =
                                    element.border.right.width + element.border.right.widthUnit
                                styleObject['border-right-style'] = element.border.right.style
                                if (element.border.right.colors.hex8) {
                                    styleObject['border-right-color'] = IDM.hex8ToRgbaString(
                                        element.border.right.colors.hex8
                                    )
                                }
                            }
                            if (element.border.bottom.width > 0) {
                                styleObject['border-bottom-width'] =
                                    element.border.bottom.width + element.border.bottom.widthUnit
                                styleObject['border-bottom-style'] = element.border.bottom.style
                                if (element.border.bottom.colors.hex8) {
                                    styleObject['border-bottom-color'] = IDM.hex8ToRgbaString(
                                        element.border.bottom.colors.hex8
                                    )
                                }
                            }
                            if (element.border.left.width > 0) {
                                styleObject['border-left-width'] =
                                    element.border.left.width + element.border.left.widthUnit
                                styleObject['border-left-style'] = element.border.left.style
                                if (element.border.left.colors.hex8) {
                                    styleObject['border-left-color'] = IDM.hex8ToRgbaString(
                                        element.border.left.colors.hex8
                                    )
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
                                titleFontObj['color'] = IDM.hex8ToRgbaString(element.fontColors.hex8)
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
                                leftFontObj['color'] = IDM.hex8ToRgbaString(element.fontColors.hex8) + ' !important'
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
                                rightFontObj['color'] = IDM.hex8ToRgbaString(element.fontColors.hex8) + ' !important'
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
                        ` #${this.moduleObject.id} .idm-navbar-right-text`
                    ]),
                    mainColorObj
                )
            }
        },
        reload() {
            //请求数据源
            this.initData()
        },
        initData() {},
        receiveBroadcastMessage(object) {
            console.log('组件收到消息', object)
        },
        setContextValue(object) {
            console.log('统一接口设置的值', object)
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
    // 背景透明
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

<template>
    <div idm-ctrl="idm_module" :id="moduleObject.id" :idm-ctrl-id="moduleObject.id" class="idm-tabbar">
        <!-- tabbar对应渲染内容 -->
        <div v-for="(item, index) in propData.tabbarList" :key="index">
            <!-- 拖拽区域 -->
            <div
                class="drag_container"
                v-show="active === index && (!item.tabSlotFunction || item.tabSlotFunction.length === 0)"
                :class="['idm-tabbar-content-container', `idm-tabbar-contentinner-${item.key}`]"
                idm-ctrl-inner
                :idm-ctrl-id="moduleObject.id"
                :idm-container-index="item.key"
            ></div>
            <!-- 自定义函数渲染 -->
            <div
                class="idm-tabbar-content-container"
                v-if="active === index && item.tabSlotFunction && item.tabSlotFunction.length > 0"
                v-html="getTabbarCustomRender(item)"
            ></div>
        </div>
        <!-- 底部tabbar -->
        <van-tabbar
            v-model="active"
            :fixed="propData.isFix"
            :placeholder="propData.isPlaceholder"
            :safe-area-inset-bottom="propData.isSafeAreaInsetBottom"
            :z-index="propData.zIndex"
            :before-change="handleTabbarBeforeChange"
            :class="[moduleObject.env === 'develop' ? 'idm-tabbar-develop' : '']"
        >
            <van-tabbar-item
                v-for="(item, index) in propData.tabbarList"
                :key="index"
                :badge="item.isShowBadge ? item.badge : ''"
                :class="{
                    'idm-tabbar-style-active-text': propData.tabbarType === 'activeText',
                    'idm-tabbar-style-active-text-active': active === index && propData.tabbarType === 'activeText'
                }"
            >
                <!-- tabbar-item插槽 -->
                <template #icon="props">
                    <div
                        class="item-center"
                        v-if="(item.icon && item.icon.length > 0) || (item.selectIcon && item.selectIcon.length > 0)"
                    >
                        <!-- 默认图标 -->
                        <svg
                            v-if="
                                (!props.active && item.icon && item.icon.length > 0) ||
                                !item.selectIcon ||
                                item.selectIcon.length == 0
                            "
                            class="idm-tabbar-icon"
                            :class="[
                                (!item.selectIcon || item.selectIcon.length == 0) && props.active
                                    ? item.isAnimate
                                        ? `idm-tabbar-select-icon ${item.animateType}`
                                        : 'idm-tabbar-select-icon'
                                    : ''
                            ]"
                            aria-hidden="true"
                        >
                            <use :xlink:href="`#${item.icon[0]}`"></use>
                        </svg>
                        <!-- 选中图标 -->
                        <svg
                            v-if="props.active && item.selectIcon && item.selectIcon.length > 0"
                            :class="
                                item.isAnimate ? `idm-tabbar-select-icon ${item.animateType}` : 'idm-tabbar-select-icon'
                            "
                            aria-hidden="true"
                        >
                            <use :xlink:href="`#${item.selectIcon[0]}`"></use>
                        </svg>
                    </div>
                    <!-- 没有设置图标的占位icon -->
                    <div
                        v-else
                        :class="
                            props.active
                                ? item.isAnimate
                                    ? `idm-tabbar-select-icon ${item.animateType}`
                                    : 'idm-tabbar-select-icon'
                                : 'idm-tabbar-icon'
                        "
                    >
                        <svg-icon icon-class="tabbar-index" v-if="index === 0"></svg-icon>
                        <svg-icon icon-class="search" v-else-if="index === 1"></svg-icon>
                        <svg-icon icon-class="tabbar-settings" v-else></svg-icon>
                    </div>
                </template>
                <!-- tabbar-item文本 -->
                <span :class="[active === index ? 'idm-tabbar-text-active' : 'idm-tabbar-text']">{{ item.tab }}</span>
                <!-- 激活文字风格下的背景 -->
                <div class="idm-tabbar-active-style-bg"></div>
            </van-tabbar-item>
        </van-tabbar>
    </div>
</template>
<script>
import Tabbar from 'vant/lib/tabbar'
import TabbarItem from 'vant/lib/tabbar-item'

export default {
    name: 'ITabbar',
    components: {
        [Tabbar.name]: Tabbar,
        [TabbarItem.name]: TabbarItem
    },
    data() {
        return {
            moduleObject: {},
            active: 0,
            propData: this.$root.propData.compositeAttr || {}
        }
    },
    created() {
        this.moduleObject = this.$root.moduleObject
        this.convertAttrToStyleObject()
        this.convertThemeListAttrToStyleObject()
        this.tabbarItemToStyleObject()
    },
    methods: {
        /**
         * 点击之前事件，返回false 可阻止切换
         * @param {index / name} index
         */
        handleTabbarBeforeChange(index) {
            if (this.propData.clickItemFunction && this.propData.clickItemFunction.length > 0) {
                const params = this.commonParam()
                try {
                    const booleanResult =
                        window[this.propData.clickItemFunction[0].name] &&
                        window[this.propData.clickItemFunction[0].name].call(this, {
                            ...params,
                            ...this.propData.clickItemFunction[0].param,
                            active: index,
                            moduleObject: this.moduleObject
                        })
                    return booleanResult
                } catch (error) {}
            }
            return true
        },
        /**
         * 设置tabbar-item角标内容
         * @param {*} item
         * @param {*} index
         * @param {*} content
         */
        handleSetBadgeContent(item, index, content) {
            if (!window.isNaN(parseInt(content))) {
                content = Number(content)
                if (content > 99) {
                    content = '99+'
                }
            }
            const tabbarList = _.cloneDeep(this.propData.tabbarList)
            tabbarList[index] = { ...item, badge: content }
            this.$set(this.propData, 'tabbarList', tabbarList)
        },
        /**
         * 获取tabbar-item角标内容
         * @param {*} item
         * @param {*} index
         */
        handleSetItemBadge(item, index) {
            switch (item.dotContentType) {
                case 'customFunction':
                    if (item.dotContentFunction && item.dotContentFunction.length > 0) {
                        const params = this.commonParam()
                        try {
                            const dotContent =
                                window[item.dotContentFunction[0].name] &&
                                window[item.dotContentFunction[0].name].call(this, {
                                    ...params,
                                    ...item.dotContentFunction[0].param,
                                    moduleObject: this.moduleObject
                                })
                            this.handleSetBadgeContent(item, index, dotContent)
                        } catch (error) {}
                    }
                    break
                case 'interface':
                    item.dotInterface &&
                        window.IDM.http
                            .get(item.dotInterface)
                            .then((res) => {
                                if (res.status == 200 && res.data.code == 200) {
                                    this.handleSetBadgeContent(item, index, res.data.data)
                                } else {
                                    IDM.message.error(res.data.message)
                                }
                            })
                            .catch(function (error) {})
                    break
            }
        },
        // 设置角标入口函数
        handleSetBadge() {
            this.propData.tabbarList &&
                this.propData.tabbarList.forEach((el, index) => {
                    if (el.isShowBadge) {
                        this.handleSetItemBadge(el, index)
                    }
                })
        },
        // 获取tabbar-item自定义渲染html
        getTabbarCustomRender(item) {
            if (item.tabSlotFunction && item.tabSlotFunction.length > 0) {
                if (!window[item.tabSlotFunction[0].name]) {
                    return this.moduleObject.env == 'develop' ? '请把动作面板打开' : ''
                }
                return (
                    window[item.tabSlotFunction[0].name] &&
                    window[item.tabSlotFunction[0].name].call(this, {
                        customParam: item.tabSlotFunction[0].param,
                        tab: item
                    })
                )
            } else {
                return '方法未找到'
            }
        },
        // 设置fix模拟高度
        handleSetFixMockHeight() {
            let height = `auto`
            if (this.moduleObject.env === 'develop' && this.propData.isFix) {
                height = IDM.develop.getDragWorkspaceInfo().height + 'px'
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-tabbar-content-container', {
                'min-height': height + ' !important'
            })
        },
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
        propDataWatchHandle(propData) {
            this.propData = propData.compositeAttr || {}
            this.convertAttrToStyleObject()
            this.convertThemeListAttrToStyleObject()
            this.tabbarItemToStyleObject()
        },
        // 设置tabbar-item样式
        tabbarItemToStyleObject() {
            this.propData.tabbarList &&
                this.propData.tabbarList.forEach((el, index) => {
                    const badgeBgColorObj = {},
                        fontObj = {},
                        selectFontObj = {},
                        iconObj = {},
                        selectIconObj = {},
                        activeBgObj = {}
                    //角标背景
                    if (el.badgeBgColor && el.badgeBgColor.hex8) {
                        badgeBgColorObj['background-color'] = IDM.hex8ToRgbaString(el.badgeBgColor.hex8)
                    }
                    // 默认字体
                    if (el.font) {
                        fontObj['font-family'] = el.font.fontFamily
                        if (el.font.fontColors.hex8) {
                            fontObj['color'] = IDM.hex8ToRgbaString(el.font.fontColors.hex8) + ' !important'
                        }
                        fontObj['font-weight'] = el.font.fontWeight && el.font.fontWeight.split(' ')[0]
                        fontObj['font-style'] = el.font.fontStyle
                        fontObj['font-size'] = el.font.fontSize + el.font.fontSizeUnit
                        fontObj['line-height'] =
                            el.font.fontLineHeight +
                            (el.font.fontLineHeightUnit == '-' ? '' : el.font.fontLineHeightUnit)
                        fontObj['text-align'] = el.font.fontTextAlign
                        fontObj['text-decoration'] = el.font.fontDecoration
                    }
                    // 选中字体
                    if (el.selectFont) {
                        selectFontObj['font-family'] = el.selectFont.fontFamily
                        if (el.selectFont.fontColors.hex8) {
                            selectFontObj['color'] = IDM.hex8ToRgbaString(el.selectFont.fontColors.hex8) + ' !important'
                        }
                        selectFontObj['font-weight'] =
                            el.selectFont.fontWeight && el.selectFont.fontWeight.split(' ')[0]
                        selectFontObj['font-style'] = el.selectFont.fontStyle
                        selectFontObj['font-size'] = el.selectFont.fontSize + el.selectFont.fontSizeUnit
                        selectFontObj['line-height'] =
                            el.selectFont.fontLineHeight +
                            (el.selectFont.fontLineHeightUnit == '-' ? '' : el.selectFont.fontLineHeightUnit)
                        selectFontObj['text-align'] = el.selectFont.fontTextAlign
                        selectFontObj['text-decoration'] = el.selectFont.fontDecoration
                    }
                    // 默认图标
                    if (el.iconSize) {
                        iconObj['width'] = el.iconSize + 'px'
                        iconObj['height'] = el.iconSize + 'px'
                        iconObj['font-size'] = el.iconSize + 'px'
                    }
                    if (el.iconColor && el.iconColor.hex8) {
                        iconObj['fill'] = IDM.hex8ToRgbaString(el.iconColor.hex8)
                        iconObj['color'] = IDM.hex8ToRgbaString(el.iconColor.hex8)
                    }
                    // 选中图标
                    if (el.selectIconSize) {
                        selectIconObj['width'] = el.selectIconSize + 'px'
                        selectIconObj['height'] = el.selectIconSize + 'px'
                        selectIconObj['font-size'] = el.selectIconSize + 'px'
                    }
                    if (el.selectIconColor && el.selectIconColor.hex8) {
                        selectIconObj['fill'] = IDM.hex8ToRgbaString(el.selectIconColor.hex8) + ' !important'
                        selectIconObj['color'] = IDM.hex8ToRgbaString(el.selectIconColor.hex8) + ' !important'
                    }
                    // 选中背景色
                    if (el.activeBgColor && el.activeBgColor.hex8) {
                        activeBgObj['background-color'] = IDM.hex8ToRgbaString(el.activeBgColor.hex8)
                    }
                    // 选中背景宽
                    if (el.activeBgWidth) {
                        activeBgObj['width'] = el.activeBgWidth
                    }
                    // 选中背景高
                    if (el.activeBgHeight) {
                        activeBgObj['height'] = el.activeBgHeight
                    }
                    // 选中背景偏移
                    if (el.activeBgTop) {
                        activeBgObj['top'] = el.activeBgTop
                    }
                    window.IDM.setStyleToPageHead(
                        this.moduleObject.id + ` .van-tabbar-item:nth-child(${index + 1}) .van-info`,
                        badgeBgColorObj
                    )
                    window.IDM.setStyleToPageHead(
                        this.moduleObject.id + ` .van-tabbar-item:nth-child(${index + 1}) .idm-tabbar-text`,
                        fontObj
                    )
                    window.IDM.setStyleToPageHead(
                        this.moduleObject.id + ` .van-tabbar-item:nth-child(${index + 1}) .idm-tabbar-text-active`,
                        selectFontObj
                    )
                    window.IDM.setStyleToPageHead(
                        this.moduleObject.id + ` .van-tabbar-item:nth-child(${index + 1}) .idm-tabbar-icon`,
                        iconObj
                    )
                    window.IDM.setStyleToPageHead(
                        this.moduleObject.id + ` .van-tabbar-item:nth-child(${index + 1}) .idm-tabbar-select-icon`,
                        selectIconObj
                    )
                    window.IDM.setStyleToPageHead(
                        this.moduleObject.id + ` .van-tabbar-item:nth-child(${index + 1}).idm-tabbar-style-active-text-active .idm-tabbar-active-style-bg`,
                        activeBgObj
                    )
                })
        },
        convertAttrToStyleObject() {
            const styleObject = {},
                fontBoxObj = {},
                activeStyleFontObj = {}
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
                        case 'width':
                        case 'height':
                            styleObject[key] = element
                            break
                        case 'activeStyleBgColor':

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
                        case 'fontBox':
                            if (element.marginTopVal) {
                                fontBoxObj['margin-top'] = element.marginTopVal
                            }
                            if (element.marginRightVal) {
                                fontBoxObj['margin-right'] = element.marginRightVal
                            }
                            if (element.marginBottomVal) {
                                fontBoxObj['margin-bottom'] = element.marginBottomVal
                            }
                            if (element.marginLeftVal) {
                                fontBoxObj['margin-left'] = element.marginLeftVal
                            }
                            if (element.paddingTopVal) {
                                fontBoxObj['padding-top'] = element.paddingTopVal
                            }
                            if (element.paddingRightVal) {
                                fontBoxObj['padding-right'] = element.paddingRightVal
                            }
                            if (element.paddingBottomVal) {
                                fontBoxObj['padding-bottom'] = element.paddingBottomVal
                            }
                            if (element.paddingLeftVal) {
                                fontBoxObj['padding-left'] = element.paddingLeftVal
                            }
                            break
                        case 'bgImgUrl':
                            styleObject['background-image'] = `url(${window.IDM.url.getWebPath(element)})`
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
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-tabbar', styleObject)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-tabbar-item__text', fontBoxObj)
            this.handleSetFixMockHeight()
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
                        ` #${this.moduleObject.id} .idm-tabbar-select-icon`,
                        ` #${this.moduleObject.id} .idm-tabbar-text-active`
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
            if (this.moduleObject.env !== 'develop' && this.propData.tabbarList.length > 0) {
                const tabbarList = _.cloneDeep(this.propData.tabbarList)
                const defaultActive = tabbarList.findIndex((el) => el.isDefaultActive)
                tabbarList.forEach((el) => {
                    el.badge = ''
                })
                this.active = defaultActive
                this.propData.tabbarList = tabbarList
            }
            this.handleSetBadge()
        },
        receiveBroadcastMessage(messageObject) {
            console.log('组件收到消息', messageObject)
            switch (messageObject.type) {
                case 'websocket':
                    if (messageObject.message) {
                        const messageData =
                            (typeof messageObject.message === 'string' && JSON.parse(messageObject.message)) ||
                            messageObject.message
                        this.propData.tabbarList.forEach((el, index) => {
                            if (messageData.includes(el.refreshKey)) {
                                this.handleSetItemBadge(el, index)
                            }
                        })
                    }
                    break
                case 'pageResize':
                    this.handleSetFixMockHeight()
                    break
            }
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
@import '~vant/lib/tabbar/index.css';
@import '~vant/lib/tabbar-item/index.css';
@import '~vant/lib/info/index.css';
@import '../style/animate.scss';
$fontSize: 12px;
.idm-tabbar {
    // tabbar背景透明
    & ::v-deep() .van-tabbar-item--active {
        background: transparent;
    }
    // 开发环境时修改定位
    & ::v-deep() .van-tabbar--fixed.idm-tabbar-develop {
        position: absolute;
    }
    .idm-tabbar-develop {
        & ::v-deep() .van-tabbar--fixed {
            position: absolute;
        }
    }
    & ::v-deep() .van-tabbar-item {
        position: relative;
    }
    .idm-tabbar-text,
    .idm-tabbar-text-active {
        font-size: $fontSize;
    }
    .item-center {
        display: flex;
        align-items: center;
    }
    .idm-tabbar-style-active-text {
        flex-direction: row;
        .idm-tabbar-text {
            display: none;
        }
    }
    .idm-tabbar-active-style-bg {
        position: absolute;
        width: 0;
        height: 80%;
        background: #d3b8b622;
        border-radius: 20px;
        left: 8%;
        top: 8%;
    }
    .idm-tabbar-style-active-text-active {
        .idm-tabbar-active-style-bg {
            transition: width 0.3s;
            width: 80%;
        }
        .idm-tabbar-text {
            display: block;
        }
        .idm-tabbar-text-active{
            font-size: 15px;
            margin: 0 0 0 10px;
        }
    }
}
</style>

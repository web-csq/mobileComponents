<template>
    <div idm-ctrl="idm_module" :id="moduleObject.id" :idm-ctrl-id="moduleObject.id" class="idm-tabbar">
        <div v-for="(item, index) in propData.tabbarList" :key="index">
            <div
                class="drag_container"
                v-show="active === index && (!item.tabSlotFunction || item.tabSlotFunction.length === 0)"
                :class="['idm-tabbar-content-container', `idm-tabbar-contentinner-${item.key}`]"
                idm-ctrl-inner
                :idm-ctrl-id="moduleObject.id"
                :idm-container-index="item.key"
            ></div>

            <div
                class="idm-tabbar-content-container"
                v-if="active === index && item.tabSlotFunction && item.tabSlotFunction.length > 0"
                v-html="getTabbarCustomRender(item)"
            ></div>
        </div>
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
            >
                <template #icon="props">
                    <div
                        class="item-center"
                        v-if="(item.icon && item.icon.length > 0) || (item.selectIcon && item.selectIcon.length > 0)"
                    >
                        <svg
                            v-if="
                                (!props.active && item.icon && item.icon.length > 0) ||
                                !item.selectIcon ||
                                item.selectIcon.length == 0
                            "
                            class="idm-tabbar-icon"
                            :class="[
                                (!item.selectIcon || item.selectIcon.length == 0) && props.active
                                    ? 'idm-tabbar-select-icon'
                                    : ''
                            ]"
                            aria-hidden="true"
                        >
                            <use :xlink:href="`#${item.icon[0]}`"></use>
                        </svg>
                        <svg
                            v-if="props.active && item.selectIcon && item.selectIcon.length > 0"
                            class="idm-tabbar-select-icon"
                            aria-hidden="true"
                        >
                            <use :xlink:href="`#${item.selectIcon[0]}`"></use>
                        </svg>
                    </div>
                    <div v-else :class="props.active ? 'idm-tabbar-select-icon' : 'idm-tabbar-icon'">
                        <svg-icon icon-class="tabbar-index" v-if="index === 0"></svg-icon>
                        <svg-icon icon-class="search" v-else-if="index === 1"></svg-icon>
                        <svg-icon icon-class="tabbar-settings" v-else></svg-icon>
                    </div>
                </template>
                <span :class="[active === index ? 'idm-tabbar-text-active' : 'idm-tabbar-text']">{{ item.tab }}</span>
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
    },
    methods: {
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
                    // 返回false 可阻止切换
                    return booleanResult
                } catch (error) {}
            }
            return true
        },
        handleSetDotContent(item, index, content) {
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
        handleSetItemDot(item, index) {
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
                            this.handleSetDotContent(item, index, dotContent)
                        } catch (error) {}
                    }
                    break
                case 'interface':
                    item.dotInterface &&
                        window.IDM.http
                            .get(item.dotInterface)
                            .then((res) => {
                                if (res.status == 200 && res.data.code == 200) {
                                    this.handleSetDotContent(item, index, res.data.data)
                                } else {
                                    IDM.message.error(res.data.message)
                                }
                            })
                            .catch(function (error) {})
                    break
            }
        },
        handleSetDot() {
            this.propData.tabbarList &&
                this.propData.tabbarList.forEach((el, index) => {
                    if (el.isShowBadge) {
                        this.handleSetItemDot(el, index)
                    }
                })
        },
        /**
         * 获取tab自定义的数据
         */
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
        propDataWatchHandle(propData) {
            this.propData = propData.compositeAttr || {}
            this.convertAttrToStyleObject()
            this.convertThemeListAttrToStyleObject()
        },
        convertAttrToStyleObject() {
            const styleObject = {},
                fontObj = {},
                selectFontObj = {},
                iconObj = {},
                selectIconObj = {},
                fontBoxObj = {}
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
                        case 'iconSize':
                            iconObj['width'] = element + 'px'
                            iconObj['height'] = element + 'px'
                            iconObj['font-size'] = element + 'px'
                            break
                        case 'iconColor':
                            iconObj['fill'] = element.hex8
                            iconObj['color'] = element.hex8
                            break
                        case 'selectIconSize':
                            selectIconObj['width'] = element + 'px'
                            selectIconObj['height'] = element + 'px'
                            selectIconObj['font-size'] = element + 'px'
                            break
                        case 'selectIconColor':
                            selectIconObj['fill'] = element.hex8 + ' !important'
                            selectIconObj['color'] = element.hex8 + ' !important'
                            break
                        case 'boxShadow':
                            styleObject['box-shadow'] = element
                            break
                        case 'width':
                        case 'height':
                            styleObject[key] = element
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
                        case 'fontBox':
                            if (element.marginTopVal) {
                                fontBoxObj['margin-top'] = `${element.marginTopVal}`
                            }
                            if (element.marginRightVal) {
                                fontBoxObj['margin-right'] = `${element.marginRightVal}`
                            }
                            if (element.marginBottomVal) {
                                fontBoxObj['margin-bottom'] = `${element.marginBottomVal}`
                            }
                            if (element.marginLeftVal) {
                                fontBoxObj['margin-left'] = `${element.marginLeftVal}`
                            }
                            if (element.paddingTopVal) {
                                fontBoxObj['padding-top'] = `${element.paddingTopVal}`
                            }
                            if (element.paddingRightVal) {
                                fontBoxObj['padding-right'] = `${element.paddingRightVal}`
                            }
                            if (element.paddingBottomVal) {
                                fontBoxObj['padding-bottom'] = `${element.paddingBottomVal}`
                            }
                            if (element.paddingLeftVal) {
                                fontBoxObj['padding-left'] = `${element.paddingLeftVal}`
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
                        case 'font':
                            fontObj['font-family'] = element.fontFamily
                            if (element.fontColors.hex8) {
                                fontObj['color'] = element.fontColors.hex8 + ' !important'
                            }
                            fontObj['font-weight'] = element.fontWeight && element.fontWeight.split(' ')[0]
                            fontObj['font-style'] = element.fontStyle
                            fontObj['font-size'] = element.fontSize + element.fontSizeUnit
                            fontObj['line-height'] =
                                element.fontLineHeight +
                                (element.fontLineHeightUnit == '-' ? '' : element.fontLineHeightUnit)
                            fontObj['text-align'] = element.fontTextAlign
                            fontObj['text-decoration'] = element.fontDecoration
                            break
                        case 'selectFont':
                            selectFontObj['font-family'] = element.fontFamily
                            if (element.fontColors.hex8) {
                                selectFontObj['color'] = element.fontColors.hex8 + ' !important'
                            }
                            selectFontObj['font-weight'] = element.fontWeight && element.fontWeight.split(' ')[0]
                            selectFontObj['font-style'] = element.fontStyle
                            selectFontObj['font-size'] = element.fontSize + element.fontSizeUnit
                            selectFontObj['line-height'] =
                                element.fontLineHeight +
                                (element.fontLineHeightUnit == '-' ? '' : element.fontLineHeightUnit)
                            selectFontObj['text-align'] = element.fontTextAlign
                            selectFontObj['text-decoration'] = element.fontDecoration
                            break
                    }
                }
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-tabbar', styleObject)
            let height = `auto`
            if (this.moduleObject.env === 'develop' && this.propData.isFix) {
                height = IDM.develop.getDragWorkspaceInfo().height + 'px'
            }
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-tabbar-content-container', {
                'min-height': height + ' !important'
            })
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-tabbar-text', fontObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-tabbar-text-active', selectFontObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-tabbar-icon', iconObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .idm-tabbar-select-icon', selectIconObj)
            window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-tabbar-item__text', fontBoxObj)
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
            // 至少2个以上
            if (this.moduleObject.env !== 'develop' && this.propData.tabbarList.length > 1) {
                const tabbarList = _.cloneDeep(this.propData.tabbarList)
                const defaultActive = tabbarList.findIndex((el) => el.isDefaultActive)
                tabbarList.forEach((el) => {
                    el.badge = ''
                })
                this.active = defaultActive
                this.propData.tabbarList = tabbarList
            }
            this.handleSetDot()
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
                                this.handleSetItemDot(el, index)
                            }
                        })
                    }
                    break
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
@import '~vant/lib/tabbar/index.css';
@import '~vant/lib/tabbar-item/index.css';
@import '~vant/lib/info/index.css';
$fontSize: 12px;
.idm-tabbar {
    & ::v-deep() .van-tabbar-item--active {
        background: transparent;
    }
    & ::v-deep() .van-tabbar--fixed.idm-tabbar-develop {
        position: absolute;
    }

    .idm-tabbar-develop {
        & ::v-deep() .van-tabbar--fixed {
            position: absolute;
        }
    }
    .idm-tabbar-text,
    .idm-tabbar-text-active {
        font-size: $fontSize;
    }
    .item-center {
        display: flex;
        align-items: center;
    }
}
</style>

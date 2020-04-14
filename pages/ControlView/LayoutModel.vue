<template lang="tea">
section.layout-model {
    div.select {
        ?: hasActiveCssSelect
        MyInput {
            title: 选择器
            v-model: cssSelect.select
            @blur: handleEditFinish
        }
    }
    div.quick-layout {
        i.icon {
            %: quickList
            :title: $it.label
            @click: handleCreateCode($_i)
            :style: {'background-image': `url(${$it.icon})` }
        }
    }
    ul.css-layout {
        ?: hasActiveCssSelect
        li.new-style {
            input {
                placeholder: 添加新样式，格式随意
                v-model: newStyle
                @keyup.enter: handleBlur
                @blur: handleBlur
            }
        }
        li.normal {
            %: layoutList
            :key: $_i + 'layout'
            div.key-value {
                i.delete {
                    @dblclick: handleDelete($it.key)
                }
                span.key {
                    ~~{{ $it.key }}
                }
                span.value {
                    ?: $it.range
                    ~~{{ cssSelect.cssom && cssSelect.cssom[$it.key] ? cssSelect.cssom[$it.key] : '' }}
                }
                input.input-value {
                    /
                    type: text
                    @blur: handleEditFinish
                    @keyup.enter: handleEditFinish
                    v-model: cssSelect.cssom[$it.key]
                }
            }
            span.range {
                v-for: (it, index) in $it.range
                :key: index + it
                @click: handleClick($it.key, it)
                |: !cssSelect.cssom || !cssSelect.cssom[$it.key] || (cssSelect.cssom && cssSelect.cssom[$it.key] && (cssSelect.cssom[$it.key] !== it)) ? true : false
                ~~{{ it }}
            }
        }
    }
}
</template>
<script>
import { cssom2List } from '@/utils/comput'
import bus from '@/utils/eventBus'
import MyInput from '@/components/MyInput'
const getLayoutModel = () => ({
    position: {
        value: '',
        range: ['relative', 'absolute', 'fixed']
    },
    display: {
        value: '',
        range: ['block', 'inline-block', 'flex', 'inline-flex', 'box', 'inline', 'none']
    },
    'align-items': {
        value: '',
        range: ['center', 'flex-start', 'flex-end']
    },
    'justify-content': {
        value: '',
        range: ['center', 'space-around', 'space-between', 'flex-start', 'flex-end']
    },
    overflow: {
        value: '',
        range: ['auto', 'hidden']
    },
    'box-sizing': { value: '', range: ['border-box'] },
    'background-repeat': { value: '', range: ['repeat', 'no-repeat'] }
})

const iconLayoutList = () => [
    {
        icon: require('@/assets/icon-flex-center-1.svg'),
        label: '完全居中',
        getCode: () => ({
            display: 'flex',
            'align-items': 'center',
            'justify-content': 'center'
        })
    },
    {
        icon: require('@/assets/icon-flex-center-2.svg'),
        label: '上下靠边',
        getCode: () => ({
            display: 'flex',
            'flex-direction': 'column',
            'justify-content': 'space-between'
        })
    },
    {
        icon: require('@/assets/icon-flex-center-3.svg'),
        label: '左右靠边',
        getCode: () => ({
            display: 'flex',
            'align-items': 'center',
            'justify-content': 'space-between'
        })
    },
    {
        icon: require('@/assets/icon-flex-column.svg'),
        label: '向下排列',
        getCode: () => ({
            display: 'flex',
            'flex-direction': 'column'
        })
    },
    {
        icon: require('@/assets/icon-wallpaper.svg'),
        label: '背景图自适应',
        getCode: () => ({
            'border-radius': '5px',
            'background-image': 'url()',
            'background-position': 'center center',
            'background-size': '100% 100%',
            'background-repeat': 'no-repeat'
        })
    },
    {
        icon: require('@/assets/icon-bgcolor.svg'),
        label: '背景色',
        getCode: () => ({
            'background-color': '#fff'
        })
    },
    {
        icon: require('@/assets/icon-top-line.svg'),
        label: '伪元素线条-上',
        getCode: () => ({
            display: 'block',
            content: `''`,
            position: 'absolute',
            left: 0,
            right: 0,
            top: 0,
            height: '1px',
            'background-color': '#000'
        })
    },
    {
        icon: require('@/assets/icon-bottom-line.svg'),
        label: '伪元素线条-下',
        getCode: () => ({
            display: 'block',
            content: `''`,
            position: 'absolute',
            left: 0,
            right: 0,
            bottom: 0,
            height: '1px',
            'background-color': '#000'
        })
    },
    {
        icon: require('@/assets/icon-left-line.svg'),
        label: '伪元素线条-左',
        getCode: () => ({
            display: 'block',
            content: `''`,
            position: 'absolute',
            left: 0,
            top: 0,
            bottom: 0,
            width: '1px',
            'background-color': '#000'
        })
    },
    {
        icon: require('@/assets/icon-right-line.svg'),
        label: '伪元素线条-右',
        getCode: () => ({
            display: 'block',
            content: `''`,
            position: 'absolute',
            top: 0,
            right: 0,
            bottom: 0,
            width: '1px',
            'background-color': '#000'
        })
    },
    {
        icon: require('@/assets/icon-border-outer.svg'),
        label: '全边框',
        getCode: () => ({
            'box-sizing': 'border-box',
            border: '1px solid red'
        })
    },
    {
        icon: require('@/assets/icon-border-clear.svg'),
        label: '无边框',
        getCode: () => ({
            border: 'none'
        })
    },
    {
        icon: require('@/assets/icon-width-height.svg'),
        label: '宽高',
        getCode: () => ({
            width: '100%',
            height: '100%'
        })
    },
    {
        icon: require('@/assets/icon-position.svg'),
        label: '相对定位',
        getCode: () => ({
            position: 'relative'
        })
    },
    {
        icon: require('@/assets/icon-flex-shrink.svg'),
        label: 'flex不压缩',
        getCode: () => ({
            'flex-shrink': 0
        })
    },
    {
        icon: require('@/assets/icon-flex-big.svg'),
        label: 'flex尽量填充',
        getCode: () => ({
            flex: 1
        })
    },
    {
        icon: require('@/assets/icon-font.svg'),
        label: '文字',
        getCode: () => ({
            'font-size': '12px',
            color: '#000'
        })
    },
    {
        icon: require('@/assets/icon-format-clear.svg'),
        label: '清除默认样式',
        getCode: () => ({
            'list-style': 'none',
            'font-style': 'normal'
        })
    },
    {
        icon: require('@/assets/icon-more-line.svg'),
        label: '多行省略',
        getCode: () => ({
            overflow: 'hidden',
            'text-overflow': 'ellipsis',
            display: '-webkit-box',
            '-webkit-line-clamp': 3,
            '-webkit-box-orient': 'vertical'
        })
    },
    {
        icon: require('@/assets/icon-button.svg'),
        label: '按钮',
        getCode: () => ({
            'background-color': '#233',
            padding: '5px 5px',
            'font-size': '15px',
            color: '#fff',
            display: 'inline-flex',
            'align-items': 'center',
            'justify-content': 'center',
            width: '60px',
            height: '30px',
            'border-radius': '5px'
        })
    }
]
export default {
    name: 'LayoutModel',
    data: () => ({
        cssSelect: [],
        quickList: iconLayoutList(),
        layoutList: [],
        newStyle: ''
    }),
    components: {
        MyInput
    },
    computed: {
        hasActiveCssSelect() {
            return this.$store.state.selectCssMid
        }
    },
    methods: {
        initLayoutList(cssSelect) {
            const cssom = JSON.parse(JSON.stringify(cssSelect.cssom)),
                layout = getLayoutModel(),
                styleObj = {}
            for (const key in cssom) {
                styleObj[key] = layout[key] ? layout[key].range : null
            }
            return cssom2List(styleObj)
        },
        handleEditFinish() {
            const element = JSON.parse(JSON.stringify(this.$store.state.editElement))
            if (!element) {
                return
            }
            const data = this.cssSelect
            for (const key in data.cssom) {
                if (data.cssom[key] == null) {
                    delete data.cssom[key] // 过滤掉所有css值为null的属性
                }
            }
            const css = element.css.map(item => (item._mid === data._mid ? data : item))
            element.css = css
            this.$store.commit('setEditElement', JSON.parse(JSON.stringify(element)))
            bus.$emit('update-element', element._mid, element)
            this.createDataLink(this.$store.state.selectCssMid)
        },
        handleClick(key, value) {
            const data = Object.assign({}, this.cssSelect)
            data.cssom[key] = value
            this.cssSelect = data
            this.handleEditFinish()
        },
        handleCreateCode(index) {
            const data = Object.assign({}, this.cssSelect)
            const _data = this.quickList[index].getCode()
            for (const key in _data) {
                data.cssom[key] = _data[key]
            }
            this.cssSelect = data
            this.handleEditFinish()
        },
        handleDelete(key) {
            this.cssSelect.cssom[key] = null
            this.handleEditFinish()
        },
        handleBlur() {
            const style = this.newStyle
            if (!style.trim().match(/[:\s]/)) {
                this.newStyle = ''
                return
            }
            let res = style.indexOf(':')
            let index = ~res ? res : style.indexOf(' ')
            const key = style.slice(0, index).trim(),
                value = style.slice(index + 1).trim()
            this.newStyle = ''
            this.handleClick(key, value) // 更新数据
        },
        createDataLink(selectCssMid) {
            if (!selectCssMid) {
                bus.$emit('change-css', null)
                return
            }
            const element = JSON.parse(JSON.stringify(this.$store.state.editElement))
            if (!element) {
                return
            }
            const [cssSelect] = element.css.filter(item => item._mid === selectCssMid)
            this.cssSelect = cssSelect
            this.layoutList = this.initLayoutList(cssSelect) // 更新数据模型
            bus.$emit('change-css', cssSelect)
        }
    },
    watch: {
        '$store.state.selectCssMid': {
            handler(value) {
                this.createDataLink(value)
            }
        }
    }
}
</script>
<style lang="stylus" scoped>
    .layout-model {
        width: 400px;
        overflow: auto;
        padding: 15px;

        .select {
            display: flex;
            align-items: center;
            height: 30px;
            width: 100%;
            margin-bottom: 30px;
            font-size: 14px;

            span {
                width: 100px;
                color: rgba(85, 41, 91, 0.8);
            }

            input {
                width: 240px;
                height: 30px;
                border: none;
                border-bottom: 1px solid rgba(85, 41, 91, 0.5);
                font-size: 16px;
            }
        }

        .new-style {
            font-size: 14px;
            margin: 15px 0;
            display: flex;
            height: 30px;

            input {
                width: 100%;
                height: 30px;
                border: none;
                font-size: 16px;
                color: rgba(85, 41, 91, 0.9);
                font-weight: bold;
                padding: 0 10px;
                border-bottom: 2px solid rgba(85, 41, 91, 0.5);

                &::placeholder {
                    font-size: 16px;
                    font-weight: normal;
                    font-style: italic;
                    color: rgba(85, 41, 91, 0.7);
                }
            }
        }

        .quick-layout {
            margin: 10px 0;
            width: 100%;

            .icon {
                width: 36px;
                height: 36px;
                display: inline-flex;
                background-repeat: no-repeat;
                background-position: center center;
                background-size: 100% 100%;
                cursor: pointer;

                &:hover {
                    background-color: rgba(85, 41, 91, 0.2);
                }
            }
        }

        .css-layout {
            > li {
                list-style: none;
            }

            .normal {
            }

            .key-value {
                font-size: 16px;
                display: flex;
                align-items: center;
                height: 25px;
                user-select: none;

                .delete {
                    cursor: pointer;
                    display: flex;
                    width: 20px;
                    height: 20px;
                    background-image: url('../../assets/icon-delete.svg');
                    background-position: center center;
                    background-size: 90% 90%;
                    background-repeat: no-repeat;
                    transform: translateX(-6px);
                }

                .key {
                    color: rgba(85, 41, 91, 1);
                    flex: 1;
                }

                .value {
                    flex: 1;
                    display: inline-flex;
                    padding: 0 8px;
                    height: 20px;
                    align-items: center;
                    border-bottom: 1px solid rgba(85, 41, 91, 0.5);
                    color: rgba(85, 41, 91, 1);
                    box-sizing: border-box;
                }

                .input-value {
                    flex: 1;
                    height: 20px;
                    padding: 0 8px;
                    font-size: 15px;
                    border: none;
                    border-bottom: 1px solid rgba(85, 41, 91, 0.5);
                }
            }

            .range {
                display: flex;
                align-items: center;
                justify-content: flex-end;
                cursor: pointer;
                color: rgba(85, 41, 91, 0.7);
                font-size: 14px;
                padding: 0 10px;

                &:hover {
                    background: rgba(85, 41, 91, 0.6);
                    color: #fff;
                }
            }
        }
    }
</style>
<template lang="tea">
    aside.element-card {
        div.select {
            span {
                @click: handleActive(2)
                :class: [{'active': active === 2}]
                ~~元素卡片
            }
        }
        ul.radio {
            |: active === 2
            li {
                VsRadio {
                    v-model: treeType
                    vs-name: wokao
                    vs-value: 1
                    ~~树状
                }
            }
            li {
                VsRadio {
                    v-model: treeType
                    vs-name: wokao
                    vs-value: 2
                    ~~瀑布
                }
            }
        }
        div.card-content {
            NewCard {
               |: active === 2
            }
        }
        
    }
</template>

<script>
import NewCard from './NewCard'
import bus from '@/utils/eventBus'
export default {
    name: 'SideBar',
    data: () => ({
        active: 2,
        treeType: 1
    }),
    components: {
        NewCard
    },
    mounted() {
        bus.$on('get-file-content', async path => {
            console.log('path', path)
        })
    },
    methods: {
        handleActive(n) {
            this.active = n
        }
    },
    watch: {
        treeType(value) {
            this.$store.commit('setTreeType', value)
        }
    }
}
</script>

<style lang="stylus" scoped>
    .element-card {
        width: 300px;
        display: flex;
        flex-direction: column;
        align-items: center;
        border-right: 1px solid rgba(85, 41, 91, 0.2);

        .card-content {
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow: auto;
        }

        .radio {
            margin: 25px;
            display: flex;
            justify-content: space-around;
            width: 100%;

            li {
                list-style: none;
            }
        }

        .select {
            width: 100%;
            height: 40px;
            display: flex;
            flex-shrink: 0;

            span {
                flex: 1;
                display: flex;
                justify-content: center;
                align-items: center;
                cursor: pointer;
                font-size: 16px;
                transition: 0.4s all ease;

                &.active {
                    background: rgba(85, 41, 91, 0.6);
                    box-shadow: 0 0 10px rgba(85, 41, 91, 0.2);
                    color: #f1f1f1;
                }
            }
        }
    }
</style>
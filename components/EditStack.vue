<template lang="tea">
    ul.stack {
        li.list {
            %: list
            span.show {
                ~~{{$it | formatAttr}}
            }
            span.icon {
                i.edit {
                    @click: handleEdit($_i)
                }
                i.up {
                    @click: handleOrder($_i, 1)
                }
                i.down {
                    @click: handleOrder($_i, 2)
                }
                i.delete {
                    @click: handleOrder($_i, 0)
                }
            }
        }
        li.add-li {
            MyInput {
                title: class名
                @keyup.enter: handleAdd
                v-model: newValue
            }
            button.add {
                @click: handleAdd
                ~~Add
            }
        }
    }
</template>

<script>
import MyInput from './MyInput'
export default {
    name: 'EditStack',
    props: {
        list: {
            type: Array,
            default: () => []
        }
    },
    components: {
        MyInput
    },
    data: () => ({
        newValue: ''
    }),
    methods: {
        handleAdd() {
            let value = ''
            const arr = this.list.concat()
            if (!this.newValue.trim()) {
                return
            }
            value = this.newValue.trim()
            arr.push(value)
            this.$emit('change', arr)
            this.newValue = ''
            return
        },
        handleOrder(i, n) {
            const arr = this.list.concat()
            const [data] = arr.splice(i, 1)
            if (n === 1) {
                // 置顶
                arr.unshift(data)
            }
            if (n === 2) {
                arr.push(data)
            }
            this.$emit('change', arr)
        },
        handleEdit(i) {
            this.newValue = this.list[i]
        }
    },
    filters: {
        formatAttr(value) {
            if (typeof value === 'object') {
                const [data] = Object.entries(value)
                return data[1] == null ? `${data[0]}` : `${data[0]}="${data[1]}"`
            }
            return value
        }
    }
}
</script>

<style lang="stylus" scoped>
.stack {
    display: flex;
    flex-direction: column;
    flex: 1;

    li {
        list-style: none;
        min-height: 26px;
        display: flex;
        align-items: center;

        &.add-li {
            display: flex;
            flex-direction: column;
            width: 100%;
            margin-top: 10px;
        }

        &.list {
            cursor: pointer;
            padding: 0 10px;
            display: flex;
            justify-content: space-between;

            &:hover {
                background: rgba(85, 41, 91, 0.1);
            }

            .show {
                color: rgba(85, 41, 91, 0.7);
                font-size: 14px;
                font-style: italic;
            }

            .icon {
                display: flex;
                align-items: center;

                i {
                    display: inline-block;
                    width: 20px;
                    height: 20px;
                    background-repeat: no-repeat;
                    background-position: center center;
                    background-size: 80% 80%;

                    &.edit {
                        background-image: url('../assets/icon-build.svg');
                    }

                    &.up {
                        background-image: url('../assets/icon-up.svg');
                    }

                    &.down {
                        background-image: url('../assets/icon-down.svg');
                    }

                    &.delete {
                        background-image: url('../assets/icon-delete.svg');
                    }
                }
            }
        }

        >input {
            flex: 1;
            height: 34px;
            width: 100%;
            padding: 0 10px;
            margin-bottom: 10px;
            font-size: 16px;
            color: #55295b;
            border: none;
            border-bottom: 1px solid rgba(85, 41, 91, 0.7);

            &::placeholder {
                font-size: 12px;
                color: rgba(85, 41, 91, 0.3);
                font-style: italic;
            }
        }

        .add {
            width: 100px;
            height: 34px;
            background: #55295b;
            color: #fff;
            flex-shrink: 0;
            align-self: flex-end;
            border-radius: 4px;
            margin-top: 10px;
        }
    }
}
</style>
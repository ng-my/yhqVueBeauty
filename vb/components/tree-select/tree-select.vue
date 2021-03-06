<template>
    <base-select v-model="value" :popup-container="popupContainer" :placeholder="placeholder" :search="search" :position="position" :multiple="multiple" :allow-clear="allowClear" @clear="clear" ref="select" @search="searchFn">
        <v-tree prefix-cls="ant-select-tree" :data="data" @select="select" @check="check" :multiple="multiple" :checkable="checkable" ref="tree" :async="async"></v-tree>
    </base-select>
</template>
<script>
    import baseSelect from './select';
    import vTree from '../tree';
    import { t } from '../../locale';

    export default {
        name: 'TreeSelect',
        components: { baseSelect, vTree },
        data() {
            return {
                value: [],
                checkable: this.multiple,
            };
        },
        props: {
            data: {
                type: Array,
                default: () => [],
            },
            multiple: {
                type: Boolean,
                default: false,
            },
            allowClear: {
                type: Boolean,
                default: false,
            },
            search: {
                type: Boolean,
                default: false,
            },
            popupContainer: {
                type: Function,
                default: () => document.body,
            },
            placeholder: {
                type: String,
                default: () => t('common.placeholder1'),
            },
            position: {
                type: String,
                default: 'absolute',
            },
            async: Function,
        },
        mounted() {
            if (this.multiple) {
                this.setValueByCheck(this.data);
            } else {
                this.setValueBySelect(this.data);
            }
        },
        methods: {
            setValueBySelect(data) {
                let res = false;
                for (const item of data) {
                    if (item.selected) {
                        const arr = [{ text: item.title, uid: item.clue }];
                        this.value = arr;
                        res = true;
                        break;
                    } else if (item.children) {
                        if (this.setValueBySelect(item.children)) {
                            res = true;
                            break;
                        }
                    }
                }
                return res;
            },
            setValueByCheck(data) {
                for (const item of data) {
                    if (item.checked && item.childrenCheckedStatus === 2) {
                        this.value.push({ text: item.title, uid: item.clue });
                    } else if (item.children) {
                        this.setValueByCheck(item.children);
                    }
                }
            },
            select(data) {
                if (this.multiple) return;

                const val = [];
                if (data[0]) {
                    val.push({ text: data[0].title, uid: data[0].clue });
                }
                this.value = val;
                this.$emit('select', data[0]);
                this.$refs.select.toggleDropdown();
            },
            check(data) {
                if (!this.multiple) return;
                const temp = [];
                for (let i = 0; i < data.length; i++) {
                    temp.push(data[i].clue);
                }
                const val = [];
                const res = [];
                for (let i = 0; i < data.length; i++) {
                    // 判断被选中的节点里面是否有我的父节点
                    if (temp.some(clue => clue !== data[i].clue && data[i].clue.startsWith(clue))) continue;
                    val.push({ text: data[i].title, uid: data[i].clue });
                    res.push(data[i]);
                }
                this.value = val;
                this.$emit('check', res);
            },
            clear(value) {
                const uids = [];
                for (const item of value) {
                    uids.push(item.uid);
                }
                this.resetTreeData(uids, this.data);
                this.$emit('clear', null);
            },
            resetTreeData(uids, data) {
                for (let i = 0; i < data.length; i++) {
                    const index = uids.indexOf(data[i].clue);
                    if (index !== -1) {
                        if (this.multiple) {
                            this.setCheck(data[i].clue);
                        } else {
                            data[i].selected = false;
                        }
                        uids.splice(index, 1);
                    }
                    if (data[i].children) {
                        this.resetTreeData(uids, data[i].children);
                    }
                }
            },
            formatClue(index) {
                let myClue = index;
                if (index < 10) {
                    myClue = `00${index}`;
                } else if (index < 100) {
                    myClue = `0${index}`;
                }
                return myClue;
            },
            fillTree(node) {
                for (let i = 0; i < node.$children.length; i++) {
                    const nodeItem = node.$children[i];
                    const clueArr = nodeItem.clue.split('-');
                    const clueLast = clueArr[clueArr.length - 1] / 1;
                    if (clueLast !== i) {
                        node.$children.splice(i, 0, { clue: `${node.clue}-${this.formatClue(i)}` });
                        i--;
                    }
                }
                return node;
            },
            setCheck(clue) {
                const route = clue.split('-');
                let node = this.$refs.tree;
                let i = 1;
                for (; i < route.length - 1; i++) {
                    this.fillTree(node);
                    node = node.$children[(route[i] / 1)];
                }
                const item = node.data[(route[i] / 1)];
                node.setCheck(item.disabled, (route[i] / 1));
            },
            searchFn(val) {
                console.log(val);
            },
        },
    };
</script>

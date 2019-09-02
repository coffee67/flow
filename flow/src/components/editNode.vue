<template>
    <el-form :model="node" ref="dataForm" label-width="100px" class="flowEditForm margin13" size="mini">
        <el-form-item label="GUID">
            <el-input disabled v-model="node.id"></el-input>
        </el-form-item>
        <el-form-item label="名称">
            <el-input v-model="node.label"></el-input>
        </el-form-item>
        <el-form-item label="活动类型">
            <el-select v-model="node.Type" placeholder="请选择">
                <el-option v-for="item in TypeList"
                           :key="item.Value"
                           :label="item.Text"
                           :value="item.Value">
                </el-option>
            </el-select>
        </el-form-item>
        <el-form-item label="备注">
            <el-input v-model="node.Remark" type="textarea" :autosize="{ minRows: 2, maxRows: 4}"></el-input>
        </el-form-item>
    </el-form>
</template>

<script>
    export default {
        name:'editNode',
        data() {
            return {
                node: {},
                TypeList: [{ Value: 1, Text: '起点' }, { Value: 2, Text: '终点' }, { Value: 3, Text: '人工活动' }, { Value: 4, Text: '自由活动' }],
            }
        },
        methods: {
            init(data, id) {
                data.nodeList.filter((node) => {
                    if (node.id === id) {
                        this.node = node;
                    }
                })
            },
        }
    }
</script>

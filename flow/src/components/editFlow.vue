<template>
    <el-form :model="flowData" ref="dataForm" label-width="100px" class="flowEditForm margin13" size="mini">
        <el-form-item label="GUID">
            <el-input disabled v-model="flowData.Id"></el-input>
        </el-form-item>
        <el-form-item label="流程名称">
            <el-input v-model="flowData.Name"></el-input>
        </el-form-item>
        <el-form-item label="备注">
            <el-input v-model="flowData.Remark" type="textarea" :autosize="{ minRows: 2, maxRows: 4}"></el-input>
        </el-form-item>
    </el-form>
</template>

<script>
    export default {
        name:'editFlow',
        data() {
            return {
                flowData: {},
            }
        },
        methods: {
            init(data) {
                this.flowData = data;
            },
        }
    }
</script>

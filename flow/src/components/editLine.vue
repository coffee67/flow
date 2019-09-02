<template>
    <el-form :model="line" ref="dataForm" label-width="100px" class="flowEditForm margin13" size="mini">
        <el-form-item label="GUID">
            <el-input disabled v-model="line.id"></el-input>
        </el-form-item>
        <el-form-item label="源活动">
            <el-input disabled v-model="line.from"></el-input>
        </el-form-item>
        <el-form-item label="目标活动">
            <el-input disabled v-model="line.to"></el-input>
        </el-form-item>
        <el-form-item label="名称">
            <el-input v-model="line.label"></el-input>
        </el-form-item>
        <el-form-item label="备注说明">
            <el-input v-model="line.Remark" type="textarea" :autosize="{ minRows: 2, maxRows: 4}"></el-input>
        </el-form-item>
    </el-form>
</template>

<script>
    export default {
        name:'editLine',
        data() {
            return {
                line: { label: '' },
                isUpdate: false,
            }
        },
        watch: {
            line: {
                handler(val) {
                    if (this.isUpdate) {
                        this.$emit('line-save', this.line);
                    }
                },
                deep: true
            }
        },
        methods: {
            init(item) {
                this.isUpdate = false;
                this.line = item;
                this.$nextTick(function () {
                    this.isUpdate = true;
                });
            },
        }
    }
</script>

<template>
    <div>
        <div style="margin-left: 50%;margin-top: 5%">
            <el-button type="primary"  size="100px" @click="add">点击注册</el-button>
        </div>
        <div>
            <router-link :to="{path:'/'}">
                <el-button type="info" style="margin-top: 10%;margin-left: 60%;" size="100px">已注册？立即登录</el-button>
            </router-link>
        </div>
        <el-dialog
                title="注册"
                :visible.sync="centerDialogVisible"
                width="30%"
                center>

            <el-form ref="form" :rules="rules" :model="form" label-width="80px">
                <el-form-item label="账号" prop="no">
                    <el-col :span="20">
                        <el-input v-model="form.no"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="名字" prop="name">
                    <el-col :span="20">
                        <el-input v-model="form.name"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-col :span="20">
                        <el-input v-model="form.password"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="年龄" prop="age">
                    <el-col :span="20">
                        <el-input v-model="form.age"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="性别">
                    <el-radio-group v-model="form.sex">
                        <el-radio label="1">男</el-radio>
                        <el-radio label="0">女</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="电话" prop="phone">
                    <el-col :span="20">
                        <el-input v-model="form.phone"></el-input>
                    </el-col>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
    <el-button @click="centerDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="save">确 定</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: "AdminManage",
    data() {
        let checkAge = (rule, value, callback) => {
            if (value > 150) {
                callback(new Error('年龄输入过大'));
            } else {
                callback();
            }
        };
        let checkDuplicate = (rule, value, callback) => {
            if (this.form.id) {
                return callback();
            }
            this.$axios.get(this.$httpUrl + "/user/findByNo?no=" + this.form.no).then(res => res.data).then(res => {
                if (res.code != 200) {

                    callback()
                } else {
                    callback(new Error('账号已经存在'));
                }
            })
        };

        return {
            tableData: [],
            pageSize: 10,
            pageNum: 1,
            total: 0,
            name: '',
            sex: '',
            sexs: [
                {
                    value: '1',
                    label: '男'
                }, {
                    value: '0',
                    label: '女'
                }
            ],
            centerDialogVisible: false,
            form: {
                id: '',
                no: '',
                name: '',
                password: '',
                age: '',
                phone: '',
                sex: '0',
                roleId: '2'
            },
            rules: {
                no: [
                    {required: true, message: '请输入账号', trigger: 'blur'},
                    {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'},
                    {validator: checkDuplicate, trigger: 'blur'}
                ],
                name: [
                    {required: true, message: '请输入名字', trigger: 'blur'}
                ],
                password: [
                    {required: true, message: '请输入密码', trigger: 'blur'},
                    {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'}
                ],
                age: [
                    {required: true, message: '请输入年龄', trigger: 'blur'},
                    {min: 1, max: 3, message: '长度在 1 到 3 个位', trigger: 'blur'},
                    {pattern: /^([1-9][0-9]*){1,3}$/, message: '年龄必须为正整数字', trigger: "blur"},
                    {validator: checkAge, trigger: 'blur'}
                ],
                phone: [
                    {required: true, message: "手机号不能为空", trigger: "blur"},
                    {pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/, message: "请输入正确的手机号码", trigger: "blur"}
                ]
            }
        }
    },
    methods: {
        resetForm() {
            this.$refs.form.resetFields();
        },
        del(id) {
            console.log(id)

            this.$axios.get(this.$httpUrl + '/user/del?id=' + id).then(res => res.data).then(res => {
                console.log(res)
                if (res.code == 200) {

                    this.$message({
                        message: '操作成功！',
                        type: 'success'
                    });
                    this.loadPost()
                } else {
                    this.$message({
                        message: '操作失败！',
                        type: 'error'
                    });
                }

            })
        },
        mod(row) {
            console.log(row)

            this.centerDialogVisible = true
            this.$nextTick(() => {
                //赋值到表单
                this.form.id = row.id
                this.form.no = row.no
                this.form.name = row.name
                this.form.password = ''
                this.form.age = row.age + ''
                this.form.sex = row.sex + ''
                this.form.phone = row.phone
                this.form.roleId = row.roleId
            })
        },
        add() {

            this.centerDialogVisible = true
            this.$nextTick(() => {
                this.resetForm()
            })

        },
        doSave() {
            this.$axios.post(this.$httpUrl + '/user/save', this.form).then(res => res.data).then(res => {
                console.log(res)
                if (res.code == 200) {

                    this.$message({
                        message: '操作成功！',
                        type: 'success'
                    });
                    this.centerDialogVisible = false
                    this.loadPost()
                    this.resetForm()
                } else {
                    this.$message({
                        message: '操作失败！',
                        type: 'error'
                    });
                }

            })
        },
        doMod() {
            this.$axios.post(this.$httpUrl + '/user/update', this.form).then(res => res.data).then(res => {
                console.log(res)
                if (res.code == 200) {

                    this.$message({
                        message: '操作成功！',
                        type: 'success'
                    });
                    this.centerDialogVisible = false
                    this.loadPost()
                    this.resetForm()
                } else {
                    this.$message({
                        message: '操作失败！',
                        type: 'error'
                    });
                }

            })
        },
        save() {
            this.$refs.form.validate((valid) => {
                if (valid) {
                    if (this.form.id) {
                        this.doMod();
                    } else {
                        this.doSave();
                    }
                } else {
                    console.log('error submit!!');
                    return false;
                }
            });

        },
        handleSizeChange(val) {
            console.log(`每页 ${val} 条`);
            this.pageNum = 1
            this.pageSize = val
            this.loadPost()
        },
        handleCurrentChange(val) {
            console.log(`当前页: ${val}`);
            this.pageNum = val
            this.loadPost()
        },
        loadGet() {
            this.$axios.get(this.$httpUrl + '/user/list').then(res => res.data).then(res => {
                console.log(res)
            })
        },
        resetParam() {
            this.name = ''
            this.sex = ''
        },
        loadPost() {
            this.$axios.post(this.$httpUrl + '/user/listPageC1', {
                pageSize: this.pageSize,
                pageNum: this.pageNum,
                param: {
                    name: this.name,
                    sex: this.sex,
                    roleId: '1'
                }
            }).then(res => res.data).then(res => {
                console.log(res)
                if (res.code == 200) {
                    this.tableData = res.data
                    this.total = res.total
                } else {
                    alert('获取数据失败')
                }

            })
        }
    },
    beforeMount() {
        //this.loadGet();
        this.loadPost()
    }
}
</script>

<style scoped>

</style>
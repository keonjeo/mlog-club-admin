
<template>
    <section>

        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.name" placeholder="名称"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="list">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>


        <el-table :data="results" highlight-current-row border v-loading="listLoading"
                  style="width: 100%;" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"></el-table-column>
            <el-table-column prop="id" label="编号"></el-table-column>

                <el-table-column prop="userId" label="userId"></el-table-column>

                <el-table-column prop="content" label="content"></el-table-column>

                <el-table-column prop="type" label="type"></el-table-column>

                <el-table-column prop="extraData" label="extraData"></el-table-column>

                <el-table-column prop="createTime" label="createTime"></el-table-column>

            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                </template>
            </el-table-column>
        </el-table>


        <el-col :span="24" class="toolbar">
            <el-pagination layout="total, sizes, prev, pager, next, jumper" :page-sizes="[20, 50, 100, 300]"
                           @current-change="handlePageChange"
                           @size-change="handleLimitChange"
                           :current-page="page.page"
                           :page-size="page.limit"
                           :total="page.total"
                           style="float:right;">
            </el-pagination>
        </el-col>


        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">

                    <el-form-item label="userId" prop="rule">
                        <el-input v-model="addForm.userId"></el-input>
                    </el-form-item>

                    <el-form-item label="content" prop="rule">
                        <el-input v-model="addForm.content"></el-input>
                    </el-form-item>

                    <el-form-item label="type" prop="rule">
                        <el-input v-model="addForm.type"></el-input>
                    </el-form-item>

                    <el-form-item label="extraData" prop="rule">
                        <el-input v-model="addForm.extraData"></el-input>
                    </el-form-item>

                    <el-form-item label="createTime" prop="rule">
                        <el-input v-model="addForm.createTime"></el-input>
                    </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>


        <el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-input v-model="editForm.id" type="hidden"></el-input>

                    <el-form-item label="userId" prop="rule">
                        <el-input v-model="editForm.userId"></el-input>
                    </el-form-item>

                    <el-form-item label="content" prop="rule">
                        <el-input v-model="editForm.content"></el-input>
                    </el-form-item>

                    <el-form-item label="type" prop="rule">
                        <el-input v-model="editForm.type"></el-input>
                    </el-form-item>

                    <el-form-item label="extraData" prop="rule">
                        <el-input v-model="editForm.extraData"></el-input>
                    </el-form-item>

                    <el-form-item label="createTime" prop="rule">
                        <el-input v-model="editForm.createTime"></el-input>
                    </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
import HttpClient from '@/apis/HttpClient';

export default {
  name: 'List',
  data() {
    return {
      results: [],
      listLoading: false,
      page: {},
      filters: {},
      selectedRows: [],

      addForm: {

        userId: '',

        content: '',

        type: '',

        extraData: '',

        createTime: '',

      },
      addFormVisible: false,
      addFormRules: {},
      addLoading: false,

      editForm: {
        id: '',

        userId: '',

        content: '',

        type: '',

        extraData: '',

        createTime: '',

      },
      editFormVisible: false,
      editFormRules: {},
      editLoading: false,
    };
  },
  mounted() {
    this.list();
  },
  methods: {
    list() {
      const me = this;
      me.listLoading = true;
      const params = Object.assign(me.filters, {
        page: me.page.page,
        limit: me.page.limit,
      });
      HttpClient.post('/api/admin/message/list', params)
        .then((data) => {
          me.results = data.results;
          me.page = data.page;
        })
        .finally(() => {
          me.listLoading = false;
        });
    },
    handlePageChange(val) {
      this.page.page = val;
      this.list();
    },
    handleLimitChange(val) {
      this.page.limit = val;
      this.list();
    },
    handleAdd() {
      this.addForm = {
        name: '',
        description: '',
      };
      this.addFormVisible = true;
    },
    addSubmit() {
      const me = this;
      HttpClient.post('/api/admin/message/create', this.addForm)
        .then((data) => {
          me.$message({ message: '提交成功', type: 'success' });
          me.addFormVisible = false;
          me.list();
        })
        .catch((rsp) => {
          me.$notify.error({ title: '错误', message: rsp.message });
        });
    },
    handleEdit(index, row) {
      const me = this;
      HttpClient.get(`/api/admin/message/${row.id}`)
        .then((data) => {
          me.editForm = Object.assign({}, data);
          me.editFormVisible = true;
        })
        .catch((rsp) => {
          me.$notify.error({ title: '错误', message: rsp.message });
        });
    },
    editSubmit() {
      const me = this;
      HttpClient.post('/api/admin/message/update', me.editForm)
        .then((data) => {
          me.list();
          me.editFormVisible = false;
        })
        .catch((rsp) => {
          me.$notify.error({ title: '错误', message: rsp.message });
        });
    },

    handleSelectionChange(val) {
      this.selectedRows = val;
    },
  },
};
</script>

<style scoped>

</style>

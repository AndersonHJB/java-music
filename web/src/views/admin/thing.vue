<template>
  <div>
    <!--页面区域-->
    <div class="page-view">
      <div class="table-operations">
        <a-space>
          <a-button type="primary" @click="handleAdd">新增</a-button>
          <a-button @click="handleBatchDelete">批量删除</a-button>
          <a-input-search addon-before="名称" enter-button @search="onSearch" @change="onSearchChange" />
        </a-space>
      </div>
      <a-table
        size="middle"
        rowKey="id"
        :loading="data.loading"
        :columns="columns"
        :data-source="data.dataList"
        :scroll="{ x: 'max-content' }"
        :row-selection="rowSelection"
        :pagination="{
          size: 'default',
          current: data.page,
          pageSize: data.pageSize,
          onChange: (current) => (data.page = current),
          showSizeChanger: false,
          showTotal: (total) => `共${total}条数据`,
        }"
      >
        <template #bodyCell="{ text, record, index, column }">
          <template v-if="column.key === 'operation'">
            <span>
              <a @click="handleEdit(record)">编辑</a>
              <a-divider type="vertical" />
              <a-popconfirm title="确定删除?" ok-text="是" cancel-text="否" @confirm="confirmDelete(record)">
                <a href="#">删除</a>
              </a-popconfirm>
            </span>
          </template>
        </template>
      </a-table>
    </div>

    <!--弹窗区域-->
    <div>
      <a-modal
        :visible="modal.visile"
        :forceRender="true"
        :title="modal.title"
        width="880px"
        ok-text="确认"
        cancel-text="取消"
        @cancel="handleCancel"
        @ok="handleOk"
      >
        <div>
          <a-form ref="myform" :label-col="{ style: { width: '80px' } }" :model="modal.form" :rules="modal.rules">
            <a-row :gutter="24">
              <a-col span="24">
                <a-form-item label="音乐名称" name="title">
                  <a-input placeholder="请输入" v-model:value="modal.form.title" />
                </a-form-item>
              </a-col>
              <a-col span="24">
                <a-form-item label="分类" name="classificationId">
                  <a-select
                    placeholder="请选择"
                    allowClear
                    :options="modal.cData"
                    :field-names="{ label: 'title', value: 'id' }"
                    v-model:value="modal.form.classificationId"
                  />
                </a-form-item>
              </a-col>
              <a-col span="24">
                <a-form-item label="封面图">
                  <a-upload-dragger
                    name="file"
                    accept="image/*"
                    :multiple="false"
                    :before-upload="beforeUpload"
                    v-model:file-list="fileList"
                  >
                    <p class="ant-upload-drag-icon">
                      <template v-if="modal.form.coverUrl">
                        <img :src="modal.form.coverUrl" style="width: 60px; height: 60px; object-fit: cover" />
                      </template>
                      <template v-else>
                        <file-image-outlined />
                      </template>
                    </p>
                    <p class="ant-upload-text"> 请选择要上传的图片 </p>
                  </a-upload-dragger>
                </a-form-item>
              </a-col>
              
              <a-col span="24">
                <a-form-item label="源文件">
                  <a-upload-dragger
                    name="file"
                    accept="audio/*"
                    :multiple="false"
                    :before-upload="beforeUpload1"
                    v-model:file-list="fileList1"
                  >
                    <p class="ant-upload-drag-icon">
                      <file-image-outlined />
                    </p>
                    <p class="ant-upload-text"> 请选择要上传的文件 </p>
                  </a-upload-dragger>
                </a-form-item>
              </a-col>

              <a-col span="24">
                <a-form-item label="音乐简介">
                  <a-textarea placeholder="请输入" v-model:value="modal.form.description" />
                </a-form-item>
              </a-col>
              <a-col span="12">
                <a-form-item label="歌手" name="geshou">
                  <a-input placeholder="请输入" v-model:value="modal.form.geshou" style="width: 100%" />
                </a-form-item>
              </a-col>
              <a-col span="12">
                <a-form-item label="语言" name="yuyan">
                  <a-select placeholder="请选择" allowClear v-model:value="modal.form.yuyan">
                    <a-select-option key="国语" value="国语">国语</a-select-option>
                    <a-select-option key="粤语" value="粤语">粤语</a-select-option>
                    <a-select-option key="英语" value="英语">英语</a-select-option>
                    <a-select-option key="法语" value="法语">法语</a-select-option>
                    <a-select-option key="日语" value="日语">日语</a-select-option>
                    <a-select-option key="西班牙语" value="西班牙语">西班牙语</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
              <a-col span="12">
                <a-form-item label="专辑" name="zhuanji">
                  <a-input placeholder="请输入" v-model:value="modal.form.zhuanji" style="width: 100%" />
                </a-form-item>
              </a-col>
              <a-col span="12">
                <a-form-item label="发行方" name="faxingfang">
                  <a-input placeholder="请输入" v-model:value="modal.form.faxingfang" style="width: 100%" />
                </a-form-item>
              </a-col>
              <a-col span="12">
                <a-form-item label="状态" name="status">
                  <a-select placeholder="请选择" allowClear v-model:value="modal.form.status">
                    <a-select-option key="0" value="0">上架</a-select-option>
                    <a-select-option key="1" value="1">下架</a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>
        </div>
      </a-modal>
    </div>
  </div>
</template>

<script setup>
  import { message } from 'ant-design-vue';
  import { createApi, listApi, updateApi, deleteApi } from '/@/api/thing';
  import { listApi as listClassificationApi } from '/@/api/classification';
  import { BASE_URL } from '/@/store/constants';
  import { FileImageOutlined } from '@ant-design/icons-vue';




  const columns = reactive([
    {
      title: '序号',
      dataIndex: 'index',
      key: 'index',
      width: 60,
    },
    {
      title: '名称',
      dataIndex: 'title',
      key: 'title',
    },
    {
      title: '歌手',
      dataIndex: 'geshou',
      key: 'geshou',
    },
    {
      title: '语言',
      dataIndex: 'yuyan',
      key: 'yuyan',
    },
    {
      title: '专辑',
      dataIndex: 'zhuanji',
      key: 'zhuanji',
    },
    {
      title: '发行方',
      dataIndex: 'faxingfang',
      key: 'faxingfang',
    },
    {
      title: '简介',
      dataIndex: 'description',
      key: 'description',
      customRender: ({ text, record, index, column }) => (text ? text.substring(0, 10) + '...' : '--'),
    },
    {
      title: '状态',
      dataIndex: 'status',
      key: 'status',
      customRender: ({ text, record, index, column }) => (text === '0' ? '上架' : '下架'),
    },
    {
      title: '操作',
      dataIndex: 'action',
      key: 'operation',
      align: 'center',
      fixed: 'right',
      width: 140,
    },
  ]);

  const beforeUpload = (file) => {
    // 改文件名
    const fileName = new Date().getTime().toString() + '.' + file.type.substring(6);
    const copyFile = new File([file], fileName);
    console.log(copyFile);
    modal.form.imageFile = copyFile;
    return false;
  };

  const beforeUpload1 = (file) => {
    // 改文件名
    const fileName = new Date().getTime().toString() + '.' + file.type.substring(6);
    const copyFile = new File([file], fileName);
    console.log(copyFile);
    modal.form.imageFile1 = copyFile;
    return false;
  };

  // 文件列表
  const fileList = ref([]);
  // 文件列表
  const fileList1 = ref([]);

  // 页面数据
  const data = reactive({
    dataList: [],
    loading: false,
    keyword: '',
    selectedRowKeys: [],
    pageSize: 10,
    page: 1,
  });

  // 弹窗数据源
  const modal = reactive({
    visile: false,
    editFlag: false,
    title: '',
    cData: [],
    form: {
      id: undefined,
      title: undefined,
      classificationId: undefined,
      geshou: undefined,
      yuyan: undefined,
      zhuanji: undefined,
      faxingfang: undefined,
      status: undefined,
      cover: undefined,
      source: undefined,
      coverUrl: undefined,
      imageFile: undefined,
      imageFile1: undefined,
    },
    rules: {
      title: [{ required: true, message: '请输入名称', trigger: 'change' }],
      classificationId: [{ required: true, message: '请选择分类', trigger: 'change' }],
      yuyan: [{ required: true, message: '请选择', trigger: 'change' }],
      status: [{ required: true, message: '请选择状态', trigger: 'change' }],
    },
  });

  const myform = ref();

  onMounted(() => {
    getDataList();
    getCDataList();
  });

  const getDataList = () => {
    data.loading = true;
    listApi({
      keyword: data.keyword,
    })
      .then((res) => {
        data.loading = false;
        console.log(res);
        res.data.forEach((item, index) => {
          item.index = index + 1;
        });
        data.dataList = res.data;
      })
      .catch((err) => {
        data.loading = false;
        console.log(err);
      });
  };

  const getCDataList = () => {
    listClassificationApi({}).then((res) => {
      modal.cData = res.data;
    });
  };


  const onSearchChange = (e) => {
    data.keyword = e.target.value;
    console.log(data.keyword);
  };

  const onSearch = () => {
    getDataList();
  };

  const rowSelection = ref({
    onChange: (selectedRowKeys, selectedRows) => {
      console.log(`selectedRowKeys: ${selectedRowKeys}`, 'selectedRows: ', selectedRows);
      data.selectedRowKeys = selectedRowKeys;
    },
  });

  const handleAdd = () => {
    resetModal();
    modal.visile = true;
    modal.editFlag = false;
    modal.title = '新增';
    // 重置
    for (const key in modal.form) {
      modal.form[key] = undefined;
    }
    modal.form.cover = undefined;
    modal.form.source = undefined;
  };
  const handleEdit = (record) => {
    resetModal();
    modal.visile = true;
    modal.editFlag = true;
    modal.title = '编辑';
    // 重置
    for (const key in modal.form) {
      modal.form[key] = undefined;
    }
    for (const key in record) {
      if (record[key]) {
        modal.form[key] = record[key];
      }
    }
    if (modal.form.cover) {
      modal.form.coverUrl = BASE_URL + '/api/staticfiles/image/' + modal.form.cover;
      modal.form.cover = undefined;
    }

    if (modal.form.source) {
      modal.form.source = undefined;
    }
  };

  const confirmDelete = (record) => {
    console.log('delete', record);
    deleteApi({ ids: record.id })
      .then((res) => {
        getDataList();
      })
      .catch((err) => {
        message.error(err.msg || '操作失败');
      });
  };

  const handleBatchDelete = () => {
    console.log(data.selectedRowKeys);
    if (data.selectedRowKeys.length <= 0) {
      console.log('hello');
      message.warn('请勾选删除项');
      return;
    }
    deleteApi({ ids: data.selectedRowKeys.join(',') })
      .then((res) => {
        message.success('删除成功');
        data.selectedRowKeys = [];
        getDataList();
      })
      .catch((err) => {
        message.error(err.msg || '操作失败');
      });
  };

  const handleOk = () => {
    myform.value
      ?.validate()
      .then(() => {
        const formData = new FormData();
        if (modal.editFlag) {
          formData.append('id', modal.form.id);
        }
        formData.append('title', modal.form.title);
        if (modal.form.classificationId) {
          formData.append('classificationId', modal.form.classificationId);
        }
        if (modal.form.imageFile) {
          formData.append('imageFile', modal.form.imageFile);
        }
        if (modal.form.imageFile1) {
          formData.append('imageFile1', modal.form.imageFile1);
        }
        formData.append('description', modal.form.description || '');
        formData.append('geshou', modal.form.geshou || '');
        formData.append('yuyan', modal.form.yuyan || '');
        formData.append('zhuanji', modal.form.zhuanji || '');
        formData.append('faxingfang', modal.form.faxingfang || '');

        if (modal.form.status) {
          formData.append('status', modal.form.status);
        }
        if (modal.editFlag) {
          updateApi(
            formData,
          )
            .then((res) => {
              hideModal();
              getDataList();
            })
            .catch((err) => {
              console.log(err);
              message.error(err.msg || '操作失败');
            });
        } else {
          createApi(formData)
            .then((res) => {
              hideModal();
              getDataList();
            })
            .catch((err) => {
              console.log(err);
              message.error(err.msg || '操作失败');
            });
        }
      })
      .catch((err) => {
        console.log('不能为空');
      });
  };

  const handleCancel = () => {
    hideModal();
  };

  // 恢复表单初始状态
  const resetModal = () => {
    myform.value?.resetFields();
    fileList.value = [];
    fileList1.value = [];
  };

  // 关闭弹窗
  const hideModal = () => {
    modal.visile = false;
  };
</script>

<style scoped lang="less">
  .page-view {
    min-height: 100%;
    background: #fff;
    padding: 24px;
    display: flex;
    flex-direction: column;
  }

  .table-operations {
    margin-bottom: 16px;
    text-align: right;
  }

  .table-operations > button {
    margin-right: 8px;
  }
</style>

<template>
  <div class="container">
    <Breadcrumb :items="['menu.log', 'menu.log.login']"></Breadcrumb>
    <a-card :title="$t('menu.log.login')" class="general-card">
      <a-row>
        <a-col :flex="1">
          <a-form
            :label-col-props="{ span: 6 }"
            :model="formModel"
            :wrapper-col-props="{ span: 18 }"
            label-align="left"
          >
            <a-row :gutter="16">
              <a-col :span="8">
                <a-form-item
                  :label="$t('log.login.form.username')"
                  field="username"
                >
                  <a-input
                    v-model="formModel.username"
                    :placeholder="$t('log.login.form.username.placeholder')"
                  />
                </a-form-item>
              </a-col>
              <a-col :span="8">
                <a-form-item :label="$t('log.login.form.ip')" field="ip">
                  <a-input
                    v-model="formModel.ip"
                    :placeholder="$t('log.login.form.ip.placeholder')"
                  />
                </a-form-item>
              </a-col>
              <a-col :span="8">
                <a-form-item
                  :label="$t('log.login.form.status')"
                  field="status"
                >
                  <a-select
                    v-model="formModel.status"
                    :options="statusOptions"
                    :placeholder="$t('log.login.form.selectDefault')"
                    allow-clear
                    @clear="resetStatus"
                  />
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>
        </a-col>
        <a-divider direction="vertical" style="height: 84px" />
        <a-col :flex="'86px'" style="text-align: right">
          <a-space :size="18" direction="vertical">
            <a-button type="primary" @click="search">
              <template #icon>
                <icon-search />
              </template>
              {{ $t('log.login.form.search') }}
            </a-button>
            <a-button @click="reset">
              <template #icon>
                <icon-refresh />
              </template>
              {{ $t('log.login.form.reset') }}
            </a-button>
          </a-space>
        </a-col>
      </a-row>
      <a-table
        :bordered="false"
        :columns="(cloneColumns as TableColumnData[])"
        :data="renderData"
        :loading="loading"
        :pagination="pagination"
        :size="size"
        row-key="id"
        @page-change="onPageChange"
      >
        <template #index="{ rowIndex }">
          {{ rowIndex + 1 }}
        </template>
        <template #status="{ record }">
          <a-badge v-if="record.status === 1" status="success" />
          <a-badge v-else status="danger" />
          {{ $t(`log.login.form.status.${record.status}`) }}
        </template>
      </a-table>
    </a-card>
  </div>
</template>

<script lang="ts" setup>
  import useLoading from '@/hooks/loading';
  import { Pagination } from '@/types/global';
  import { computed, reactive, ref, watch } from 'vue';
  import { SelectOptionData, TableColumnData } from '@arco-design/web-vue';
  import { LoginLogParams, LoginLogRecord, queryLoginLogList } from '@/api/log';
  import { useI18n } from 'vue-i18n';
  import { cloneDeep } from 'lodash';

  type Column = TableColumnData & { checked?: true };
  type SizeProps = 'mini' | 'small' | 'medium' | 'large';
  const generateFormModel = () => {
    return {
      username: undefined,
      ip: undefined,
      status: undefined,
    };
  };

  const { t } = useI18n();
  const { loading, setLoading } = useLoading(true);
  const cloneColumns = ref<Column[]>([]);
  const showColumns = ref<Column[]>([]);
  const renderData = ref<LoginLogRecord[]>([]);
  const formModel = ref(generateFormModel());
  const size = ref<SizeProps>('medium');
  const basePagination: Pagination = {
    current: 1,
    pageSize: 20,
  };
  const pagination = reactive({
    ...basePagination,
  });
  const columns = computed<TableColumnData[]>(() => [
    {
      title: t('log.login.columns.index'),
      dataIndex: 'index',
      slotName: 'index',
    },
    {
      title: t('log.login.columns.username'),
      dataIndex: 'username',
      slotName: 'username',
    },
    {
      title: t('log.login.columns.ip'),
      dataIndex: 'ip',
      slotName: 'ip',
    },
    {
      title: t('log.login.columns.browser'),
      dataIndex: 'browser',
      slotName: 'browser',
    },
    {
      title: t('log.login.columns.device'),
      dataIndex: 'device',
      slotName: 'device',
    },
    {
      title: t('log.login.columns.city'),
      dataIndex: 'city',
      slotName: 'city',
    },
    {
      title: t('log.login.columns.status'),
      dataIndex: 'status',
      slotName: 'status',
    },
    {
      title: t('log.login.columns.msg'),
      dataIndex: 'msg',
      slotName: 'msg',
    },
    {
      title: t('log.login.columns.created_time'),
      dataIndex: 'created_time',
      slotName: 'created_time',
    },
  ]);
  const statusOptions = computed<SelectOptionData[]>(() => [
    {
      label: t('log.login.form.status.1'),
      value: 1,
    },
    {
      label: t('log.login.form.status.0'),
      value: 0,
    },
  ]);

  // 请求数据
  const fetchData = async (params: LoginLogParams = { page: 1, size: 20 }) => {
    setLoading(true);
    try {
      const { data } = await queryLoginLogList(params);
      renderData.value = data.items;
      pagination.total = data.total;
      pagination.current = params.page;
    } catch (error) {
      // console.log(error);
    } finally {
      setLoading(false);
    }
  };
  fetchData();
  // 事件: 分页
  const onPageChange = (current: number) => {
    fetchData({ page: current, size: basePagination.pageSize });
  };

  // 事件: 搜索
  const search = () => {
    fetchData({
      ...basePagination,
      ...formModel.value,
    } as unknown as LoginLogParams);
  };

  // 事件: 重置
  const reset = () => {
    formModel.value = generateFormModel();
  };

  // 事件: 重置状态
  const resetStatus = () => {
    formModel.value.status = undefined;
  };

  // 监听columns变化
  watch(
    () => columns.value,
    (val) => {
      cloneColumns.value = cloneDeep(val);
      cloneColumns.value.forEach((item, index) => {
        item.checked = true;
      });
      showColumns.value = cloneDeep(cloneColumns.value);
    },
    { deep: true, immediate: true }
  );
</script>

<script lang="ts">
  export default {
    name: 'Login',
  };
</script>

<style lang="less" scoped></style>
<template>
  <a-spin style="display: block" :loading="loading">
    <a-tabs v-model:activeKey="messageType" type="rounded" destroy-on-hide>
      <a-tab-pane v-for="item in tabList" :key="item.key">
        <template #title>
          <span> {{ item.title }}{{ formatUnreadLength(item.key) }} </span>
        </template>
        <a-result v-if="!renderList.length" status="404">
          <template #subtitle> {{ $t('messageBox.noContent') }} </template>
        </a-result>
        <List
          :render-list="renderList"
          :unread-count="unreadCount"
          @item-click="handleItemClick"
        />
      </a-tab-pane>
      <!-- <template #extra>
        <a-button type="text" @click="emptyList">
          {{ $t('messageBox.tab.button') }}
        </a-button>
      </template> -->
    </a-tabs>
  </a-spin>
</template>

<script setup>
  import { ref, reactive, toRefs, computed, provide } from 'vue';
  import { useI18n } from 'vue-i18n';
  import {
    queryMessageList,
    setMessageStatus,
    // MessageRecord,
    // MessageListType,
  } from '@/api/message';
  import useLoading from '@/hooks/loading';
  import List from './list.vue';
  const { loading, setLoading } = useLoading(true);
  const messageType = ref('message');

  const { t } = useI18n();
  const messageData = reactive({
    renderList: [],
    messageList: [],
  });
  toRefs(messageData);
  const tabList = [
    {
      key: 'message',
      title: t('messageBox.tab.title.message'),
    },
    {
      key: 'notice',
      title: t('messageBox.tab.title.notice'),
    },
    {
      key: 'todo',
      title: t('messageBox.tab.title.todo'),
    },
  ];
  async function fetchSourceData() {
    setLoading(true);
    try {
      const { data } = await queryMessageList();
      messageData.messageList = data;
    } catch (err) {
      // you can report use errorHandler or other
    } finally {
      setLoading(false);
    }
  }
  async function readMessage(list) {
    const ids = list.map((item) => item.id);
    const {data } = await setMessageStatus({ ids });
    fetchSourceData();
  }
  const renderList = computed(() => {
    return messageData.messageList.filter(
      (item) => messageType.value === item.type
    );
  });
  const unreadCount = computed(() => {
    return renderList.value.filter((item) => !item.status).length;
  });
  const getUnreadList = (type) => {
    const list = messageData.messageList.filter(
      (item) => item.type === type && !item.status
    );
    return list;
  };
  const formatUnreadLength = (type) => {
    const list = getUnreadList(type);
    return list.length ? `(${list.length})` : ``;
  };
  const handleItemClick = (items) => {
    if (renderList.value.length) readMessage([...items]);
  };
  const emptyList = () => {
    messageData.messageList = [];
  };
  fetchSourceData();

</script>

<style scoped lang="less">
  :deep(.arco-popover-popup-content) {
    padding: 0;
  }

  :deep(.arco-list-item-meta) {
    align-items: flex-start;
  }
  :deep(.arco-tabs-nav) {
    padding: 14px 0 12px 16px;
    border-bottom: 1px solid var(--color-neutral-3);
  }
  :deep(.arco-tabs-content) {
    padding-top: 0;
    .arco-result-subtitle {
      color: rgb(var(--gray-6));
    }
  }
</style>

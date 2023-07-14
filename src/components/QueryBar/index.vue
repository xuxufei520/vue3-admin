<template>
  <a-card :bordered="false" id="query-bar-card">
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <slot name="extendRow"></slot>
        <a-row :gutter="24">
          <a-col
            style="padding-left: 0"
            :sm="24"
            :md="12"
            :lg="8"
            :xl="6"
            :xxl="6"
            v-for="(item, index) in props.labelList"
            :key="index"
          >
            <a-form-item :label="item.includes('hide') ? null : item" style="margin-bottom: 32px">
              <slot :name="item"></slot>
            </a-form-item>
          </a-col>
          <slot name="extendCol"></slot>
          <a-col
            v-if="showBtn"
            :sm="24"
            :md="12"
            :lg="8"
            :xl="6"
            :xxl="6"
            class="float"
            :style="{ transform: translateXStr }"
          >
            <a-form-item>
              <span class="querybar_submit_buttons">
                <slot name="buttons">
                  <a-button @click="refreshQuery">{{ props.refreshQueryBtnText }}</a-button>
                  <a-button type="primary" @click="query" :loading="loading">{{
                    props.queryBtnText
                  }}</a-button>
                </slot>
              </span>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </div>
  </a-card>
</template>
<script lang="ts" setup>
  import { watch, nextTick, onMounted, ref, onUnmounted } from 'vue';

  interface Props {
    labelList: any; // label名称集合[]
    labelMinWidth?: number; // 默认label样式支持5个字的宽度(85) 如果超出则传入label宽度
    showBtn?: boolean; // 支持权限控制按钮是否显示
    loading?: boolean;
    queryBtnText?: string;
    refreshQueryBtnText?: string;
  }
  const props = withDefaults(defineProps<Props>(), {
    labelList: [],
    labelMinWidth: 85,
    showBtn: true,
    loading: false,
    queryBtnText: '查询',
    refreshQueryBtnText: '重置',
  });
  const emits = defineEmits<{
    (_event: 'query'): void;
    (_event: 'refresh-query'): void;
  }>();
  const setLabelWidth = () => {
    const main: any = document.getElementById('query-bar-card');
    const target = main.getElementsByTagName('label');
    Array.from(target).forEach((item: any) => {
      item.style.display = 'block';
      item.style.minWidth = props.labelMinWidth + 'px';
    });
  };
  const translateXStr = ref<string>('translateX(0)');
  const setBtnPosition = () => {
    const width = document.body.offsetWidth;
    const length = props.labelList.length;
    let times = 0;
    if (width > 1600) {
      // xxl
      times = 4 - (length % 4) - 1;
    } else if (width > 1200) {
      // xl
      times = 4 - (length % 4) - 1;
    } else if (width > 992) {
      // lg
      times = 3 - (length % 3) - 1;
    } else if (width > 768) {
      // md
      times = 2 - (length % 2) - 1;
    } else if (width > 576) {
      // sm
      times = 0;
    }
    translateXStr.value = `translateX(${times}00%)`;
  };
  const query = () => {
    emits('query');
  };
  const refreshQuery = () => {
    emits('refresh-query');
  };
  watch(
    () => props.labelList,
    async () => {
      // 切换labelList重置label
      await nextTick();
      setLabelWidth();
      setBtnPosition();
    },
    {
      immediate: true,
    },
  );

  onMounted(() => {
    window.onresize = () => {
      setLabelWidth();
      setBtnPosition();
    };
  });
  onUnmounted(() => {
    window.onresize = null;
  });
</script>
<style lang="less" scoped>
  #query-bar-card {
    margin-bottom: 14px;

    :deep(.ant-card-body) {
      padding: 32px;
      padding-bottom: 0;
    }

    .table-page-search-wrapper {
      .ant-form-inline {
        display: block;

        .ant-form-item {
          display: flex;
          margin-right: 0;
          margin-bottom: 24px;

          > :deep(.ant-form-item-label) {
            width: auto;
            padding-right: 8px;
            line-height: 32px;
          }

          :deep(.ant-form-item-control-wrapper) {
            display: inline-block;
            flex: 1 1;
            overflow: hidden;
            vertical-align: middle;

            .ant-form-item-control {
              height: 32px;
              line-height: 32px;

              @media screen and (width <= 576px) {
                :deep(.ant-select) {
                  max-width: 293px;
                }
              }

              @media screen and (width >= 576px) and (width <= 768px) {
                .ant-select {
                  max-width: 521px;
                }
              }

              @media screen and (width >= 768px) and (width <= 992px) {
                // 一行2个
                .ant-select {
                  max-width: 216px;
                }
              }

              @media screen and (width >= 1000px) and (width <= 1200px) {
                // 一行3个
                .ant-select {
                  max-width: 175px;
                }
              }

              @media screen and (width >= 1200px) and (width <= 1500px) {
                // 一行3个
                .ant-select {
                  max-width: 175px;
                }
              }

              @media screen and (width >= 1500px) and (width <= 1600px) {
                .ant-select {
                  max-width: 212px;
                }
              }

              @media screen and (width >= 1600px) {
                .ant-select {
                  max-width: 291px;
                }
              }
            }
          }
        }

        .float {
          // float: right;
          .querybar_submit_buttons {
            display: flex;
            box-sizing: border-box;
            justify-content: flex-end;

            :deep(button) {
              margin-left: 16px;

              &:first-child {
                margin-left: 0;
              }
            }
          }
        }
      }
    }
  }
</style>

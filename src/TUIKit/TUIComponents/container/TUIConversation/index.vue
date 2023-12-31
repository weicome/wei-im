<template>
  <div class="TUI-conversation">
    <div class="network" v-if="isNetwork">
      <i class="icon icon-error">!</i>
      <p>️{{ $t("TUIConversation.网络异常，请您检查网络设置") }}</p>
    </div>
    <main class="TUI-conversation-list">
      <TUIConversationList
        :currentID="currentConversationID"
        :data="conversationData"
        @handleItem="handleCurrentConversation"
        :isH5="env.isH5"
        :displayOnlineStatus="displayOnlineStatus"
        :userStatusList="userStatusList"
      />
    </main>
  </div>
</template>
<script lang="ts">
import { defineComponent, reactive, toRefs, computed, watch } from "vue";
import TUIConversationList from "./components/list";
import { caculateTimeago, isArrayEqual } from "../utils";
import {
  handleAvatar,
  handleName,
  handleShowLastMessage,
  handleAt,
} from "../TUIChat/utils/utils";

const TUIConversation = defineComponent({
  name: "TUIConversation",

  components: {
    TUIConversationList,
  },
  props: {
    displayOnlineStatus: {
      type: Boolean,
      default: false,
    },
    ToConversationID: {
      type: Number,
      default: 0,
    },
  },
  setup(props: any, ctx: any) {
    const TUIServer: any = TUIConversation?.TUIServer;
    const data = reactive({
      currentConversationID: "",
      conversationData: {
        list: [],
        handleItemAvator: (item: any) => handleAvatar(item),
        handleItemName: (item: any) => handleName(item),
        handleShowAt: (item: any) => handleAt(item),
        handleShowMessage: (item: any) => handleShowLastMessage(item),
        handleItemTime: (time: number) => {
          if (time > 0) {
            return caculateTimeago(time * 1000);
          }
          return "";
        },
      },
      userIDList: [],
      netWork: "",
      env: TUIServer.TUICore.TUIEnv,
      displayOnlineStatus: false,
      userStatusList: new Map(),
    });

    TUIServer.bind(data);
    TUIConversationList.TUIServer = TUIServer;

    watch(
      () => data.currentConversationID,
      (newVal: any) => {
        ctx.emit("current", newVal);
      },
      {
        deep: true,
      }
    );
    watch(
      () => data.conversationData.list,
      (newVal: Array<any>) => {
        if (newVal.length > 0) {
          newVal.forEach((item: any) => {
            if (item.userProfile.userID == props.ToConversationID) {
              ctx.emit("current", item.conversationID);
              handleCurrentConversation(item);
            }
          });
        }
      },
      {
        deep: true,
      }
    );
    watch(
      () => props.displayOnlineStatus,
      async (newVal: any, oldVal: any) => {
        if (newVal === oldVal) return;
        data.displayOnlineStatus = newVal;
        TUIServer.TUICore.TUIServer.TUIContact.handleUserStatus(
          data.displayOnlineStatus,
          [...data.userIDList]
        );
      },
      { immediate: true }
    );

    watch(
      () => [...data.userIDList],
      async (newVal: any, oldVal: any) => {
        if (isArrayEqual(newVal, oldVal)) return;
        TUIServer.TUICore.TUIServer.TUIContact.handleUserStatus(
          data.displayOnlineStatus,
          [...data.userIDList]
        );
      },
      {
        deep: true,
      }
    );

    const isNetwork = computed(() => {
      const disconnected =
        data.netWork === TUIServer.TUICore.TIM.TYPES.NET_STATE_DISCONNECTED;
      const connecting =
        data.netWork === TUIServer.TUICore.TIM.TYPES.NET_STATE_CONNECTING;
      return disconnected || connecting;
    });

    const handleCurrentConversation = (value: any) => {
      TUIServer.handleCurrentConversation(value);
    };

    return {
      ...toRefs(data),
      handleCurrentConversation,
      isNetwork,
    };
  },
});
export default TUIConversation;
</script>

<style scoped lang="scss" src="./style/index.scss"></style>

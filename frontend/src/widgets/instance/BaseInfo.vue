<script setup lang="ts">
import { ref } from "vue";
import type { LayoutCard } from "@/types";
import { useLayoutCardTools } from "../../hooks/useCardTools";
import { onMounted, computed } from "vue";
import { t } from "@/lang/i18n";
import { useInstanceInfo } from "@/hooks/useInstance";
import { CheckCircleOutlined, ExclamationCircleOutlined } from "@ant-design/icons-vue";
import { GLOBAL_INSTANCE_NAME } from "../../config/const";
import { parseTimestamp } from "../../tools/time";
import { dockerPortsArray } from "@/tools/common";
import DockerInfo from "./dialogs/DockerInfo.vue";

const props = defineProps<{
  card: LayoutCard;
}>();

const DockerInfoDialog = ref<InstanceType<typeof DockerInfo>>();
const { getMetaOrRouteValue } = useLayoutCardTools(props.card);

const instanceId = getMetaOrRouteValue("instanceId");
const daemonId = getMetaOrRouteValue("daemonId");

const { statusText, isRunning, isStopped, instanceTypeText, instanceInfo, execute } =
  useInstanceInfo({
    instanceId,
    daemonId,
    autoRefresh: true
  });

const getInstanceName = computed(() => {
  if (instanceInfo.value?.config.nickname === GLOBAL_INSTANCE_NAME) {
    return t("TXT_CODE_5bdaf23d");
  } else {
    return instanceInfo.value?.config.nickname;
  }
});

const instanceGameServerInfo = computed(() => {
  if (instanceInfo.value?.info?.mcPingOnline) {
    return {
      players: `${instanceInfo.value?.info.currentPlayers} / ${instanceInfo.value?.info.maxPlayers}`,
      version: instanceInfo.value?.info.version
    };
  } else {
    return null;
  }
});

onMounted(async () => {
  if (instanceId && daemonId) {
    await execute({
      params: {
        uuid: instanceId,
        daemonId: daemonId
      }
    });
  }
});
</script>

<template>
  <!-- eslint-disable vue/html-indent -->
  <CardPanel class="containerWrapper" style="height: 100%">
    <template #title>
      {{ card.title }}
    </template>
    <template #body>
      <a-typography-paragraph>
        <div class="flex flex-wrap instance-tag">
          <!-- instance status -->
          <a-tag v-if="isRunning" color="green" class="tag">
            <CheckCircleOutlined />
            {{ statusText }}
          </a-tag>
          <a-tag v-else-if="isStopped" class="tag">
            <ExclamationCircleOutlined />
            {{ statusText }}
          </a-tag>
          <a-tag v-else class="tag" color="pink">
            {{ statusText }}
          </a-tag>
        </div>
      </a-typography-paragraph>
      <a-typography-paragraph>
        {{ t("TXT_CODE_7ec9c59c") }}{{ getInstanceName }}
      </a-typography-paragraph>

      <a-typography-paragraph v-if="instanceGameServerInfo">
        <span>{{ t("TXT_CODE_855c4a1c") }}</span>
        <span>{{ instanceGameServerInfo.players }}</span>
      </a-typography-paragraph>
      <a-typography-paragraph v-if="instanceGameServerInfo">
        <span>
          {{ t("TXT_CODE_e260a220") }}
        </span>
        <span>
          {{ instanceGameServerInfo.version }}
        </span>
      </a-typography-paragraph>

      <a-typography-paragraph>
        {{ t("TXT_CODE_46f575ae") }}{{ parseTimestamp(instanceInfo?.config.lastDatetime) }}
      </a-typography-paragraph>
      <a-typography-paragraph v-if="instanceInfo?.config.processType === 'docker'">
        {{ t("TXT_CODE_4f917a65") }}
        <a href="javascript:;" @click="DockerInfoDialog?.openDialog()">
          {{ t("TXT_CODE_530f5951") }}
        </a>
      </a-typography-paragraph>

      <a-typography-paragraph
        v-if="
          instanceInfo?.config.processType === 'docker' &&
          Number(instanceInfo?.config.docker.ports?.length) > 0
        "
      >
        {{ t("TXT_CODE_2e4469f6") }}
        <div style="padding: 10px 0px 0px 16px">
          <div
            v-for="(item, index) in dockerPortsArray(instanceInfo?.config.docker.ports ?? [])"
            :key="index"
            style="margin-bottom: 2px"
          >
            <span>{{ t("TXT_CODE_8dfc41ef") }}: {{ item.host }}</span>
            <span style="margin-left: 6px">{{ t("TXT_CODE_8f8103b7") }}: {{ item.container }}</span>
            <span style="margin-left: 8px">
              <a-tag color="green">{{ item.protocol.toUpperCase() }}</a-tag>
            </span>
          </div>
        </div>
      </a-typography-paragraph>
      <a-typography-paragraph>
        <span>{{ t("TXT_CODE_ae747cc0") }}</span>
        <span>{{ parseTimestamp(instanceInfo?.config.endTime) || t("TXT_CODE_e3a77a77") }}</span>
      </a-typography-paragraph>
      <a-typography-paragraph v-if="!instanceGameServerInfo">
        {{ t("TXT_CODE_8b8e08a6") }}{{ parseTimestamp(instanceInfo?.config.createDatetime) }}
      </a-typography-paragraph>
    </template>
  </CardPanel>

  <DockerInfo ref="DockerInfoDialog" :docker-info="instanceInfo?.config.docker" />
</template>

<style lang="scss" scoped>
.instance-tag {
  margin-left: -4px;
  margin-right: -4px;
  .tag {
    margin: 4px;
  }
}
</style>

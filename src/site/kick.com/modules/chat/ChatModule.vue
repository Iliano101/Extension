<template>
	<Suspense>
		<ChatController ref="controller" :channel-id="channelID" :slug="slug" />
	</Suspense>
</template>

<script setup lang="ts">
import { ref, watch } from "vue";
import { Logger } from "@/common/Logger";
import { useStaticRenderFunctionHook } from "@/common/ReactHooks";
import { useChannelContext } from "@/composable/channel/useChannelContext";
import { declareModule } from "@/composable/useModule";
import ChatController from "./ChatController.vue";

const { markAsReady } = declareModule<"KICK">("chat", {
	name: "Chat",
	depends_on: [],
});

const slug = ref("");
const channelID = ref("");
useChannelContext(channelID.value);
watch(
	slug,
	async (v) => {
		if (!v) return;
		const resp = await fetch(`https://kick.com/api/v2/channels/${v}`).catch((err) => {
			Logger.Get().error("failed to fetch channel data", err);
		});
		if (!resp) return;
		const { user_id: id } = await resp.json();
		if (!id) return;
		channelID.value = id.toString() as string;
	},
	{ immediate: true },
);

useStaticRenderFunctionHook<{ channelSlug: string }>("#chatroom-messages", 2, function (this, old, props, ref) {
	slug.value = props.channelSlug;
	return old ? old.call(this, props, ref) : null;
});

markAsReady();
</script>

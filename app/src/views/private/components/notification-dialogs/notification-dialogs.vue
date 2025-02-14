<template>
	<div class="notification-dialogs">
		<v-dialog :active="true" v-for="notification in notifications" :key="notification.id" persist>
			<v-card :class="[notification.type]">
				<v-card-title>{{ notification.title }}</v-card-title>
				<v-card-text v-if="notification.text">
					{{ notification.text }}

					<v-error v-if="notification.error" :error="notification.error" />
				</v-card-text>
				<v-card-actions>
					<v-button
						secondary
						v-if="notification.type === 'error' && admin && notification.code === 'UNKNOWN'"
					>
						<a target="_blank" :href="getGitHubIssueLink(notification.id, notification)">
							{{ $t('report_error') }}
						</a>
					</v-button>
					<v-button @click="done(notification.id)">{{ $t('dismiss') }}</v-button>
				</v-card-actions>
			</v-card>
		</v-dialog>
	</div>
</template>

<script lang="ts">
import { defineComponent, computed, ref, watch } from '@vue/composition-api';
import SidebarButton from '../sidebar-button';
import NotificationItem from '../notification-item';
import { useNotificationsStore, useUserStore } from '@/stores/';
import router from '@/router';
import { Notification } from '@/types';
import { useProjectInfo } from '@/modules/settings/composables/use-project-info';

export default defineComponent({
	components: { SidebarButton, NotificationItem },
	setup(props) {
		const { parsedInfo } = useProjectInfo();
		const notificationsStore = useNotificationsStore();
		const userStore = useUserStore();

		const notifications = computed(() => notificationsStore.state.dialogs);

		return { notifications, admin: userStore.isAdmin, done, getGitHubIssueLink };

		function getGitHubIssueLink(id: string, notification: Notification) {
			const debugInfo = `<!-- Please put a detailed explanation of the problem here. -->

---

### Project details
Directus Version: ${parsedInfo.value?.directus.version}
Environment: ${process.env.NODE_ENV}
OS: ${parsedInfo.value?.os.type} ${parsedInfo.value?.os.version}
Node: ${parsedInfo.value?.node.version}

### Error

Title: ${notification.title || 'none'}
Message: ${notification.text || 'none'}

<details>
<summary>Stack Trace</summary>
<pre>
${JSON.stringify(notification.error, Object.getOwnPropertyNames(notification.error), 2)}
</pre>
</details>
			`;

			return `https://github.com/directus/next/issues/new?body=${encodeURIComponent(debugInfo)}`;
		}

		function done(id: string) {
			notificationsStore.remove(id);
		}
	},
});
</script>

<style lang="scss" scoped>
.notification-dialogs {
	position: relative;
}

.v-error {
	margin-top: 12px;
}
</style>

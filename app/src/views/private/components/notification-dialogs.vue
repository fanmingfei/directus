<script setup lang="ts">
import { useNotificationsStore } from '@/stores/notifications';
import { useUserStore } from '@/stores/user';
import { Snackbar } from '@/types/notifications';
import { computed } from 'vue';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();

const notificationsStore = useNotificationsStore();
const { isAdmin } = useUserStore();

const notifications = computed(() => notificationsStore.dialogs);

const done = async (notification: Snackbar) => {
	if (notification.dismissAction) {
		await notification.dismissAction();
	}

	notificationsStore.remove(notification.id);
};
</script>

<template>
	<div class="notification-dialogs">
		<v-dialog v-for="notification in notifications" :key="notification.id" model-value persist>
			<v-card :class="[notification.type]">
				<v-card-title>{{ notification.title }}</v-card-title>
				<v-card-text v-if="notification.text || notification.error">
					{{ notification.text }}

					<v-error v-if="notification.error" :error="notification.error" />
				</v-card-text>
				<v-card-actions>
					<v-button v-if="notification.type === 'error' && isAdmin && notification.code === 'UNKNOWN'" secondary>
						<a target="_blank" href="https://github.com/directus/directus/issues/new?template=bug_report.yml">
							{{ t('report_error') }}
						</a>
					</v-button>
					<v-button @click="done(notification)">{{ notification.dismissText ?? t('dismiss') }}</v-button>
				</v-card-actions>
			</v-card>
		</v-dialog>
	</div>
</template>

<style lang="scss" scoped>
.notification-dialogs {
	position: relative;
}

.v-error {
	margin-top: 12px;
}
</style>

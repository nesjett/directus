<template>
	<div ref="wrapper" class="v-date-picker">
		<input class="input" type="text" :placeholder="t('enter_a_value')" data-input />
	</div>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, ref, onMounted, onBeforeUnmount, computed, PropType, watch } from 'vue';
import Flatpickr from 'flatpickr';
import { format, formatISO } from 'date-fns';
import { getFlatpickrLocale } from '@/utils/get-flatpickr-locale';

export default defineComponent({
	props: {
		modelValue: {
			type: String,
			default: null,
		},
		disabled: {
			type: Boolean,
			default: false,
		},
		type: {
			type: String as PropType<'timestamp' | 'dateTime' | 'time' | 'date'>,
			required: true,
			validator: (val: string) => ['dateTime', 'date', 'time', 'timestamp'].includes(val),
		},
		includeSeconds: {
			type: Boolean,
			default: false,
		},
		use24: {
			type: Boolean,
			default: true,
		},
	},
	emits: ['update:modelValue'],
	setup(props, { emit }) {
		const { t } = useI18n();

		const wrapper = ref<HTMLElement | null>(null);
		let flatpickr: Flatpickr.Instance | null;

		onMounted(async () => {
			if (wrapper.value) {
				const flatpickrLocale = await getFlatpickrLocale();
				flatpickr = Flatpickr(wrapper.value, { ...flatpickrOptions.value, locale: flatpickrLocale });
			}

			watch(
				() => props.modelValue,
				() => {
					if (props.modelValue) {
						flatpickr?.setDate(props.modelValue, false);
					} else {
						flatpickr?.clear();
					}
				},
				{ immediate: true }
			);
		});

		onBeforeUnmount(() => {
			if (flatpickr) {
				const selectedDate = flatpickr.selectedDates.length > 0 ? flatpickr.selectedDates[0] : null;
				emitValue(selectedDate);

				flatpickr.destroy();
				flatpickr = null;
			}
		});

		const defaultOptions = {
			static: true,
			inline: true,
			nextArrow:
				'<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px"><path d="M10 6L8.59 7.41 13.17 12l-4.58 4.59L10 18l6-6-6-6z"/></svg>',
			prevArrow:
				'<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px"><path d="M15.41 7.41L14 6l-6 6 6 6 1.41-1.41L10.83 12l4.58-4.59z"/></svg>',
			wrap: true,

			onChange(selectedDates: Date[], _dateStr: string, _instance: Flatpickr.Instance) {
				const selectedDate = selectedDates.length > 0 ? selectedDates[0] : null;
				emitValue(selectedDate);
			},
			onReady(_selectedDates: Date[], _dateStr: string, instance: Flatpickr.Instance) {
				const setToNowButton: HTMLElement = document.createElement('button');
				setToNowButton.innerHTML = t('interfaces.datetime.set_to_now');
				setToNowButton.classList.add('set-to-now-button');
				setToNowButton.tabIndex = -1;
				setToNowButton.addEventListener('click', setToNow);
				instance.calendarContainer.appendChild(setToNowButton);
			},
		};

		const flatpickrOptions = computed<Record<string, any>>(() => {
			return Object.assign({}, defaultOptions, {
				enableSeconds: props.includeSeconds,
				enableTime: ['dateTime', 'time', 'timestamp'].includes(props.type),
				noCalendar: props.type === 'time',
				time_24hr: props.use24,
			});
		});

		function emitValue(value: Date | null) {
			if (!value) return emit('update:modelValue', null);

			switch (props.type) {
				case 'dateTime':
					return emit('update:modelValue', format(value, "yyyy-MM-dd'T'HH:mm:ss"));
				case 'date':
					return emit('update:modelValue', format(value, 'yyyy-MM-dd'));
				case 'time':
					return emit('update:modelValue', format(value, 'HH:mm:ss'));
				case 'timestamp':
					return emit('update:modelValue', formatISO(value));
			}
		}

		function setToNow() {
			flatpickr?.setDate(new Date(), true);
		}

		return { t, wrapper };
	},
});
</script>

<style>
@import 'flatpickr/dist/flatpickr.css';
@import './flatpickr-overrides.css';
</style>

<style lang="scss" scoped>
.v-date-picker {
	.input {
		display: none;
	}
}
</style>

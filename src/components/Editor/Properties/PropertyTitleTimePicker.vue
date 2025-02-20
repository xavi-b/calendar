<!--
  - @copyright Copyright (c) 2020 Georg Ehrke <oc.list@georgehrke.com>
  - @copyright Copyright (c) 2019 Jakob Röhrl <jakob.roehrl@web.de>
  -
  - @author Georg Ehrke <oc.list@georgehrke.com>
  - @author Jakob Röhrl <jakob.roehrl@web.de>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div class="property-title-time-picker">
		<div v-if="!isReadOnly"
			class="property-title-time-picker__time-pickers">
			<DatePicker :date="startDate"
				:timezone-id="startTimezone"
				prefix="from"
				:is-all-day="isAllDay"
				:append-to-body="appendToBody"
				:user-timezone-id="userTimezone"
				@change="changeStart"
				@change-timezone="changeStartTimezone" />

			<DatePicker :date="endDate"
				:timezone-id="endTimezone"
				prefix="to"
				:is-all-day="isAllDay"
				:append-to-body="appendToBody"
				:user-timezone-id="userTimezone"
				@change="changeEnd"
				@change-timezone="changeEndTimezone" />
		</div>
		<div v-if="isReadOnly"
			class="property-title-time-picker__time-pickers property-title-time-picker__time-pickers--readonly">
			<div class="property-title-time-picker-read-only-wrapper">
				<div class="property-title-time-picker-read-only-wrapper__label">
					{{ formattedStart }}
				</div>
				<IconTimezone v-if="!isAllDay"
					:title="endTimezone"
					:class="{ 'highlighted-timezone-icon': highlightStartTimezone }"
					:size="20" />
				<div class="property-title-time-picker-read-only-wrapper">
					<div class="property-title-time-picker-read-only-wrapper__label">
						{{ formattedEnd }}
					</div>
					<IconTimezone v-if="!isAllDay"
						:title="endTimezone"
						:class="{ 'highlighted-timezone-icon': highlightStartTimezone }"
						:size="20" />
				</div>
			</div>

			<div v-if="!isReadOnly" class="property-title-time-picker__all-day">
				<input id="allDay"
					:checked="isAllDay"
					type="checkbox"
					class="checkbox"
					:disabled="!canModifyAllDay"
					@change="toggleAllDay">
				<label v-tooltip="allDayTooltip"
					for="allDay">
					{{ $t('calendar', 'All day') }}
				</label>
			</div>
		</div>
	</div>
</template>

<script>
import moment from '@nextcloud/moment'
import DatePicker from '../../Shared/DatePicker.vue'
import IconTimezone from 'vue-material-design-icons/Web'
import { mapState } from 'vuex'

export default {
	name: 'PropertyTitleTimePicker',
	components: {
		DatePicker,
		IconTimezone,
	},
	props: {
		/**
		 * Whether or not the editor is viewed in read-only
		 */
		isReadOnly: {
			type: Boolean,
			required: true,
		},
		/**
		 * Start date of the event
		 */
		startDate: {
			type: Date,
			required: true,
		},
		/**
		 * Timezone of the start date
		 */
		startTimezone: {
			type: String,
			required: true,
		},
		/**
		 * End date of the event
		 */
		endDate: {
			type: Date,
			required: true,
		},
		/**
		 * Timezone of the end date
		 */
		endTimezone: {
			type: String,
			required: true,
		},
		/**
		 * Whether or not the event is all-day
		 */
		isAllDay: {
			type: Boolean,
			required: true,
		},
		/**
		 * Whether or not the user can toggle the all-day property
		 * This is set to false, whenever this event is part of a recurrence-set
		 */
		canModifyAllDay: {
			type: Boolean,
			required: true,
		},
		/**
		 * The current timezone of the user
		 * This is used to highlight if the event is in a different timezone
		 */
		userTimezone: {
			type: String,
			required: true,
		},
		/**
		 * Whether to append the datepickers to body or not.
		 * Necessary in the AppSidebar, otherwise it will be cut off be the
		 * AppSidebar edges.
		 */
		appendToBody: {
			type: Boolean,
			default: false,
		},
	},
	data() {
		return {
			showStartTimezone: false,
			showEndTimezone: false,
		}
	},
	computed: {
		...mapState({
			locale: (state) => state.settings.momentLocale,
		}),
		/**
		 * Tooltip for the All-day checkbox.
		 * If the all-day checkbox is disabled, this tooltip gives an explanation to the user
		 * why it is disabled
		 *
		 * @return {string|null}
		 */
		allDayTooltip() {
			if (this.canModifyAllDay) {
				return null
			}
			if (this.isReadOnly) {
				return null
			}

			return this.$t('calendar', 'Cannot modify all-day setting for events that are part of a recurrence-set.')
		},
		/**
		 *
		 * @return {string}
		 */
		formattedStart() {
			if (this.isAllDay) {
				return this.$t('calendar', 'from {startDate}', {
					startDate: moment(this.startDate).locale(this.locale).format('L'),
					endDate: moment(this.endDate).locale(this.locale).format('L'),
				})
			}

			return this.$t('calendar', 'from {startDate} at {startTime}', {
				startDate: moment(this.startDate).locale(this.locale).format('L'),
				startTime: moment(this.startDate).locale(this.locale).format('LT'),
			})
		},
		/**
		 *
		 * @return {string}
		 */
		formattedEnd() {
			if (this.isAllDay) {
				return this.$t('calendar', 'to {endDate}', {
					endDate: moment(this.endDate).locale(this.locale).format('L'),
				})
			}

			return this.$t('calendar', 'to {endDate} at {endTime}', {
				endDate: moment(this.endDate).locale(this.locale).format('L'),
				endTime: moment(this.endDate).locale(this.locale).format('LT'),
			})
		},
		/**
		 * @return {boolean}
		 */
		highlightStartTimezone() {
			return this.startTimezone !== this.userTimezone
		},
		/**
		 * @return {boolean}
		 */
		highlightEndTimezone() {
			return this.endTimezone !== this.userTimezone
		},
	},
	methods: {
		/**
		 * Update the start date
		 *
		 * @param {Date} value The new start date
		 */
		changeStart(value) {
			this.$emit('update-start-date', value)
		},
		/**
		 * Updates the timezone of the start date
		 *
		 * @param {string} value The new start timezone
		 */
		changeStartTimezone(value) {
			// If the value didn't change, value is null
			if (!value) {
				return
			}

			this.$emit('update-start-timezone', value)
		},
		/**
		 * Update the end date
		 *
		 * @param {Date} value The new end date
		 */
		changeEnd(value) {
			this.$emit('update-end-date', value)
		},
		/**
		 * Updates the timezone of the end date
		 *
		 * @param {string} value The new end timezone
		 */
		changeEndTimezone(value) {
			// If the value didn't change, value is null
			if (!value) {
				return
			}

			this.$emit('update-end-timezone', value)
		},
		/**
		 * Toggles the all-day state of an event
		 */
		toggleAllDay() {
			if (!this.canModifyAllDay) {
				return
			}

			this.$emit('toggle-all-day')
		},
	},
}
</script>

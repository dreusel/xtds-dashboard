<template>
	<v-container fluid>
		<div class="dashboard-page">
			<v-row no-gutters class="d-flex justify-space-between mt-10 mb-6">
				<h1 class="page-title">Dashboard</h1>
			</v-row>
			<v-row>
				<v-col cols="12">
					<v-card class="support-requests mx-1 mb-1">
						<v-card-title class="pa-6 pb-0">
							<p>Teams</p>
						</v-card-title>
						<v-card-text class="pa-0">
							<v-data-table
								v-model="selected"
								:headers="headers"
								:items="enrichedTeams"
								:search="''"
								:loading="loading"
								item-key="name"
								fixed-header
								height="100vh"
								hide-default-footer
								disable-pagination
							>
								<template v-slot:[`item.dancersSelectedInTeamPerc`]="{ item }">
									<span>{{ formatPercentage1(item.dancersSelectedInTeamPerc) }}</span>
								</template>
								<template v-slot:[`item.selectedDancersPerc`]="{ item }">
									<span>{{ formatPercentage1(item.selectedDancersPerc) }}</span>
								</template>
								<template v-slot:[`item.shiftsPerc`]="{ item }">
									<span>{{ formatPercentage2(item.shiftsPerc) }}</span>
								</template>
								<template v-slot:[`item.shiftsPerDancer`]="{ item }">
									<span>{{ item.shiftsPerDancer.toFixed(1) }}</span>
								</template>
								<template v-slot:[`item.shiftTimeClaimed`]="{ item }">
									<span>{{ formatHours(item.shiftTimeClaimed) }}</span>
								</template>
								<template v-slot:[`item.shiftTimeOfTotalPerc`]="{ item }">
									<span>{{ formatPercentage2(item.shiftTimeOfTotalPerc) }}</span>
								</template>
								<template v-slot:[`item.shiftTimePerDancer`]="{ item }">
									<span>{{ formatHours(item.shiftTimePerDancer) }}</span>
								</template>
							</v-data-table>
						</v-card-text>
					</v-card>
				</v-col>
			</v-row>
		</div>
	</v-container>
</template>

<script>
import axios from 'axios';
import state from '../../state.js';

export default {
	name: "Dashboard",
	components: {
	},
	data() {
		return {
			teams: [],
			dancers: [],
			shifts: [],
			selected: [],
			headers: [
				{ text: 'Team', value: 'name' },
				{ text: 'Registered', value: 'dancersRegistered' },
				{ text: 'Selected', value: 'dancersSelected' },
				{ text: 'Selected (%)', value: 'dancersSelectedInTeamPerc' },
				{ text: '% of total', value: 'selectedDancersPerc' },
				{ text: 'Shifts claimed', value: 'shiftsClaimed' },
				{ text: '% of total shifts', value: 'shiftsPerc' },
				{ text: 'Shifts per dancer', value: 'shiftsPerDancer' },
				{ text: 'Shift Time claimed', value: 'shiftTimeClaimed' },
				{ text: '% Shift Time of total', value: 'shiftTimeOfTotalPerc' },
				{ text: 'Shift Time per dancer', value: 'shiftTimePerDancer' },
			],
		};
	},
	computed: {
		loading: function () {
			return !this.teams.length || !this.dancers.length || !this.shifts.length || !this.enrichedTeams.length;
		},
		loggedIn: () => state.loggedIn,
		enrichedTeams: function () {
			const enrichedTeams = this.teams.map(team => ({
				...team,
				dancers: [],
				dancersRegistered: 0,
				dancersSelected: 0,
				shiftTimeClaimed: 0,
				shiftsClaimed: 0,
			}));


			const totalDancers = this.dancers.length;
			const totalDancersSelected = this.dancers.filter(d => d.status_info.status === 2).length;

			let totalShiftsTime = 0;
			let totalShifts = 0;
			// let totalShiftTimeClaimed = 0;

			const teamsById = Object.fromEntries(enrichedTeams.map(d => [d.id, d]));

			this.dancers.forEach(dancer => {
				teamsById[dancer.team.id].dancersRegistered++;
				if (dancer.status_info.status === 2) {
					teamsById[dancer.team.id].dancersSelected ++;
				}
				teamsById[dancer.team.id].dancers.push(dancer);
			});

			this.shifts.forEach(shift => {
				totalShiftsTime += (shift.duration * shift.shift_slots.length);
				totalShifts += shift.shift_slots.length;
				shift.shift_slots.forEach(slot => {
					if (slot.team) {
						console.log(`team ${slot.team.id} has shift ${shift.id} of duration ${shift.duration}`);
						teamsById[slot.team.id].shiftTimeClaimed += shift.duration;
						teamsById[slot.team.id].shiftsClaimed ++;
						// totalShiftTimeClaimed += shift.duration;
					}
				});
			});

			enrichedTeams.forEach(team => {
				team.dancersSelectedInTeamPerc = team.dancersRegistered ? (team.dancersSelected / team.dancersRegistered) : 1;
				team.totalDancersPerc = (team.dancers.length / totalDancers);
				team.selectedDancersPerc = (team.dancersSelected / totalDancersSelected);
				team.shiftTimeOfTotalPerc = team.shiftTimeClaimed / totalShiftsTime;
				team.shiftTimePerDancer = team.dancersSelected ? team.shiftTimeClaimed / team.dancersSelected : 0;
				team.shiftsPerc = team.shiftsClaimed / totalShifts;
				team.shiftsPerDancer = team.dancersSelected ? team.shiftsClaimed / team.dancersSelected : 0;
			});

			return enrichedTeams;
		}
	},
	watch: {
		async loggedIn(newValue) {
			if (newValue) {
				const [teamsResponse, dancersResponse, shiftsResponse] = await Promise.all([
					axios.get('/teams/'),
					axios.get('/contestants/'),
					axios.get('/shifts/'),
				]);
				this.teams = teamsResponse.data;
				this.dancers = dancersResponse.data;
				this.shifts = shiftsResponse.data;
				this.loading = false;
			}
		},
	},
	methods: {
		formatPercentage1(value) {
			return `${(value * 100).toFixed(1)}%`;
		},
		formatPercentage2(value) {
			return `${(value * 100).toFixed(2)}%`;
		},
		formatHours(value) {
			return `${(value / 3600).toFixed(2)}`;
		},
	},
	mounted() {
	},
};
</script>

<style src="./Dashboard.scss" lang="scss"/>

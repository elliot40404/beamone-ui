<script setup>
	import Card from '../components/Card.vue';
	import { Line, Pie } from 'vue-chartjs';
	import dayjs from 'dayjs';
	import duration from 'dayjs/plugin/duration';
	import {
		Chart as ChartJS,
		CategoryScale,
		LinearScale,
		LineElement,
		PointElement,
		ArcElement,
		Legend,
		Tooltip,
	} from 'chart.js';
	import { onMounted, ref } from 'vue';
	import { computed } from '@vue/reactivity';
	dayjs.extend(duration);
	const rateSet = () => (Math.random() * (10 - 6) + 6).toFixed(2);
	const data = ref({
		successful: 0,
		to_process: 0,
		queued: 0,
		failed: 0,
		total: 0,
		remaining: 0,
		rate: 0,
		eta: 0,
		config: {},
		pie: [0, 0],
	});
	ChartJS.register(
		CategoryScale,
		LinearScale,
		PointElement,
		LineElement,
		ArcElement,
		Legend,
		Tooltip
	);
	const chartOptions = {
		responsive: true,
	};
	const pieData = {
		labels: ['Done', 'Left'],
		datasets: [
			{
				backgroundColor: ['#A4F67D', '#79C7FF'],
				data: [0, 0],
			},
		],
	};
	const lineData = {
		labels: ['time', 'time', 'time', 'time', 'time', 'time', 'time', 'time'],
		datasets: [
			{
				label: 'URLS',
				data: [0, 0, 0, 0, 0, 0, 0, 0],
				borderColor: 'rgb(62, 167, 243)',
			},
		],
	};

	const pollingRate = ref(30);

	const fetchData = async () => {
		const req = await fetch('http://localhost:4444/stats');
		const res = await req.json();
		data.value.rate = res.remaining == 0 ? 0 : rateSet();
		data.value.eta = dayjs
			.duration((res.remaining / data.value.rate) * 60 * 1000)
			.format('HH:mm:ss');
		data.value.config = res.config;
		data.value.successful = res.successful;
		data.value.to_process = res.to_process;
		data.value.queued = res.queued;
		data.value.failed = res.failed;
		data.value.total = res.total;
		data.value.remaining = res.remaining;
		pieDataC.value = [res.successful, res.remaining];
		lineDataC.value = res.remaining;
		if (res.polling_rate && res.polling_rate !== 0) {
			pollingRate.value = res.polling_rate;
		}
	};
	const finalData = computed(() => data.value);
	const pieKey = ref(0);
	const lineKey = ref(0);
	const pieDataC = computed({
		get() {
			return pieData;
		},
		set(val) {
			pieData.datasets[0].data = val;
			pieKey.value++;
		},
	});
	const lineDataC = computed({
		get() {
			return lineData;
		},
		set(val) {
			lineData.labels.shift();
			lineData.labels.push(new Date().toLocaleString().split(',')[1].trim());
			lineData.datasets[0].data.shift();
			lineData.datasets[0].data.push(val);
			lineKey.value++;
		},
	});
	onMounted(async () => {
		await fetchData();
		setInterval(() => fetchData(), pollingRate.value * 1000);
	});
</script>

<template>
	<main>
		<header>
			<h1>BEAMONE UI</h1>
			<ion-icon @click="fetchData" name="reload-outline"></ion-icon>
		</header>
		<div class="grid">
			<Card
				title="successful"
				icon="checkmark-done-outline"
				color="#95B17F"
				:value="finalData.successful"
			/>
			<Card
				title="to process"
				icon="hourglass-outline"
				color="#038DF0"
				:value="finalData.to_process"
			/>
			<Card
				title="failed"
				icon="close-outline"
				color="#FF825A"
				:value="finalData.failed"
			/>
			<Card
				title="queued"
				icon="flag-outline"
				color="#FF0000"
				:value="finalData.queued"
			/>
			<Card class="line-chart" custom>
				<div class="graph">
					<Line
						:key="lineKey"
						:chart-options="chartOptions"
						:chart-data="lineDataC"
						css-classes="chart"
					/>
				</div>
			</Card>
			<Card custom>
				<div class="timings">
					<table>
						<tr>
							<th>ETA (mins)</th>
							<th>Rate(urls/min)</th>
						</tr>
						<tr>
							<td>~{{ finalData.eta }}</td>
							<td>{{ finalData.rate }}</td>
						</tr>
					</table>
					<table>
						<tr>
							<th>Total Links</th>
							<th>Remaing Links</th>
						</tr>
						<tr>
							<td>{{ finalData.total }}</td>
							<td>{{ finalData.remaining }}</td>
						</tr>
					</table>
				</div>
			</Card>
			<Card custom>
				<div class="graph">
					<Pie
						:key="pieKey"
						:chart-options="chartOptions"
						:chart-data="pieDataC"
						css-classes="pie"
					/>
				</div>
			</Card>
			<Card class="config-variables" custom>
				<div class="config">
					<table>
						<thead>
							<tr>
								<th>
									<ion-icon name="settings-outline"></ion-icon>
									Config Variables
								</th>
							</tr>
						</thead>
						<tbody>
							<tr v-for="x in Object.keys(data.config)" :key="x">
								<td>{{ x }}</td>
								<td>{{ data.config[x] }}</td>
							</tr>
						</tbody>
					</table>
				</div>
			</Card>
		</div>
	</main>
</template>

<style scoped lang="scss">
	header {
		display: inline-flex;
		align-items: center;
		gap: 0.8rem;
	}
	main {
		padding: 1.5rem;
		min-height: 100vh;
		margin: 0 auto;
		max-width: 1200px;
	}
	.grid {
		display: grid;
		height: calc(100vh - 100px);
		padding-top: 2rem;
		place-content: center;
		grid-template-columns: repeat(4, 1fr);
		grid-template-rows: repeat(3, 1fr);
		grid-gap: 1rem;
		@media only screen and (max-width: 900px) {
			display: flex;
			flex-direction: column;
			margin: 1rem auto;
			gap: 1rem;
			max-width: 400px;
			height: auto;
		}
	}
	.line-chart {
		grid-column: 1 / span 2;
		grid-row: span 2;
	}
	.config-variables {
		grid-column: 4 / 5;
		grid-row: -3 / -1;
	}
	.config {
		display: flex;
		justify-content: center;
		align-items: center;
		// height: 100%;
		table {
			padding: 0.8rem;
			td {
				padding: 0.5rem;
				font-size: 0.8rem;
			}
			thead > tr {
				text-align: center;
				th {
					display: flex;
					align-items: center;
					gap: 0.5rem;
					white-space: nowrap;
					text-align: center;
					font-weight: 500;
					color: rgb(69, 69, 69);
					padding: 0.5rem;
				}
			}
		}
	}
	.timings {
		height: 100%;
		padding: 0.5rem;
		table {
			height: 50%;
			width: 100%;
			padding: 0.6rem;
			th {
				font-size: 0.8rem;
				font-weight: 500;
				text-align: left;
			}
			th:nth-child(2) {
				padding-left: 0.6rem;
			}
			td {
				font-size: 1.2rem;
				font-weight: bold;
			}
			td:nth-child(2) {
				padding-left: 1rem;
			}
		}
	}
	.graph {
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.chart {
		height: 90%;
		width: 90%;
	}
	.pie {
		padding: 0.5rem;
		height: 80%;
		width: 80%;
	}
</style>

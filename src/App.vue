<template>
	<div class="currency-converter">
		<h1 class="title">Конвертер валют</h1>
		<div v-if="loading" class="loading">Loading...</div>
		<div class="content-container" v-else>
			<input v-model="value" @input="convertCurrency" class="input-field" />
			<select v-model="selectedCurrency" @change="convertCurrency" class="input-field">
				<option v-for="currency in currencies" :key="currency.NumCode" :value="currency.Value">
					{{ currency.CharCode }} - {{ currency.Name }}
				</option>
			</select>
			<div v-if="error" class="error">{{ error }}</div>
			<div v-else class="result-container">
				<input class="input-field input-mod" v-model="result" readonly />
				<span>₽</span>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import { ref, onMounted, watch } from 'vue';

interface Currency {
	NumCode: string;
	CharCode: string;
	Name: string;
	Value: number;
}

export default {
	setup() {
		const loading = ref(true);
		const error = ref<string | null>(null);
		const value = ref<number | null>(null);
		const selectedCurrency = ref<number | null>(null);
		const currencies = ref<Currency[]>([]);
		const result = ref<number | null>(null);

		const fetchCurrencies = async () => {
			try {
				const response = await fetch('https://www.cbr-xml-daily.ru/daily_json.js');
				const data = await response.json();
				currencies.value = Object.values(data.Valute) as Currency[];
			} catch (error: any) {
				error.value = 'Ошибка загрузки данных';
				console.log(error);
			} finally {
				loading.value = false;
			}
		};
		const convertCurrency = () => {
			const currency = currencies.value.find(c => c.Value === selectedCurrency.value);
			if (currency && value.value !== null) {
				const calculatedResult = (value.value * currency.Value / 10);
				result.value = +calculatedResult.toFixed(2);
			}
		};

		onMounted(() => {
			fetchCurrencies();

			const savedCurrency = localStorage.getItem('selectedCurrency');
			if (savedCurrency) {
				selectedCurrency.value = +savedCurrency;
			}
		});

		watch(selectedCurrency, newCurrency => {
			localStorage.setItem('selectedCurrency', newCurrency);
		});

		return {
			loading,
			error,
			value,
			selectedCurrency,
			currencies,
			result,
			fetchCurrencies,
			convertCurrency
		};
	}
};
</script>

<style>
.currency-converter {
	max-width: 400px;
	margin: auto;
	margin-top: 50px;
	box-shadow: 0px 5px 5px 5px grey;
	border-radius: 10px;
	padding: 20px;
}

.title {
	font-size: 28px;
}

.loading {
	font-size: 18px;
}

.content-container {
	display: flex;
	flex-direction: column;
	gap: 20px;
}

.input-field {
	font-size: 16px;
	padding: 5px;
}

.error {
	color: red;
	font-size: 16px;
	margin-bottom: 10px;
}

.result-container {
	display: flex;
	align-items: center;
	gap: 5px;
}

.input-mod {
	font-weight: bold;
	width: 100%;
}

input {
	direction: rtl;
}
</style>
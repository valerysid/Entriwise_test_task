<script>
	let isDownloading = false;

	function formatDate(inputDate) {
		const date = new Date(inputDate);
		const options = { month: 'short', day: 'numeric' };
		return date.toLocaleDateString(undefined, options);
	}

	function convertCurrency(item, xe){
		const {amount, currency} = item
		if (xe) {
			return {amount: (amount/xe.rates[currency]).toFixed(2), currency: "USD"}
		}

		return {amount, currency}
	}

	async function getXe(){
		return new Promise((resolve, reject) => {

		var requestURL = 'https://api.exchangerate.host/latest?base=usd'; 
		var request = new XMLHttpRequest(); 
		request.open('GET', requestURL);
		request.responseType = 'json';
		
		request.onload = function () {
			if (request.status === 200) {
				resolve(request.response);
			} else {
				reject(new Error(`Exchangerate error! Status: ${request.status}`));
			}
		}
		request.onerror = function () {
			reject(new Error('Exchangerate error!'));
		};
		request.send();
		})}

	function downloadCSV(orders){
		const blob = new Blob([convertToCSV(orders)], { type: 'text/csv' });
		const url = window.URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = 'data.csv';
		document.body.appendChild(a);
		a.click();
		window.URL.revokeObjectURL(url);
	}
	
	async function fetchAndDownloadData() {
	  isDownloading = true;
  
	  try {
		let orders = await fetch('https://api-staging.entriwise.com/mock/test-task-orders');
		let items = await fetch('https://api-staging.entriwise.com/mock/test-task-items');

		if (!orders.ok || !items.ok) {
		  throw new Error(`API request failed`);
		}

		let xe
		try {
			xe = await getXe()
		}
		catch(e) {
			console.log(e)
			xe = null
		}
  
		orders = await orders.json();
		items = await items.json();

		items = items.items.reduce((acc, item) => {
			acc[item.itemId] = {...item, ...convertCurrency(item, xe)}
			return acc
		}, {})
		orders = orders.orders.map(order=>({...order, ...items[order.itemId], date: formatDate(order.date)}))
  
		downloadCSV(orders)
	  } catch (error) {
		console.error(error);
	  } finally {
		isDownloading = false;
	  }
	}
  
	function convertToCSV(data) {
		const headers = Object.keys(data[0]);
		const csvRows = [];

		csvRows.push(headers.join(','));

		for (const item of data) {
			const values = headers.map(header => {
			const value = item[header];
			return (typeof value === 'string' && value.includes(',')) ? `"${value}"` : value;
			});
			csvRows.push(values.join(','));
		}

		return csvRows.join('\n');
	}
  </script>
  
  <main>
	<div class="navbar">
		<div class="logo">Logo</div>
		<div class="item-links">
		  <a class="item-link" href="#">Item 1</a>
		  <a class="item-link" href="#">Item 2</a>
		  <a class="item-link" href="#">Item 3</a>
		  <a class="item-link" href="#">Item 4</a>
		</div>
	  </div>
	  
	  <div class="grid">
		<div class="box">
		  <button
		  on:click={fetchAndDownloadData}
		  disabled={isDownloading}
		>
		  {#if isDownloading}
			<div class="spinner"></div>Loading
		  {:else}
			Download
		  {/if}
		</button>
		</div>
		<div class="box">
		</div>
		<div class="box">
		</div>
		<div class="box">
		</div>

  </main>
  
  <style>  
  .navbar {
    background-color: #365B7B;
    display: flex;
    justify-content: space-between;
	align-items: center;
    padding: 10px 30px;
	border-radius: 3px;
	box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.3);
  }

  .logo {
    color: white;
  }

  .item-links {
    display: flex;
    gap: 20px;
  }

  .item-link {
    color: white;
	text-decoration: underline;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: repeat(2, 1fr);
    gap: 20px;
    padding: 20px;
  }

  .box {
    background-color: #A0D4C6;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100px;
    color: white;
	border: 2px solid gray;
	border-radius: 4px;
  }

  button {
	background-color: #365B7B;
	color: white;
	border: none;
	border-radius: 5px;
	cursor: pointer;
	box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.3);
	width: 150px;
  }

  .spinner {
	  border: 4px solid rgba(0, 0, 0, 0.3);
	  border-top: 4px solid white;
	  border-radius: 50%;
	  width: 10px;
	  height: 10px;
	  animation: spin 1s linear infinite;
	  margin-right: 0.5rem;
	  display: inline-block;
	}
  
	@keyframes spin {
	  0% { transform: rotate(0deg); }
	  100% { transform: rotate(360deg); }
	}
  </style>
  
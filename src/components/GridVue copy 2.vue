
<template>
	<div>
					<select v-model="selectedHotelLocation">
									<option value="">Select Location</option>
									<option value="mainMakkah">Main Makkah</option>
									<option value="shiftingMakkah">Shifting Makkah</option>
									<option value="madinah">Madinah</option>
					</select>
					<br><br>
					<button @click="addRoomLine">Add Room Line</button>
					<br><br>
					<button @click="clearUnconfirmedRows">Clear Unconfirmed Rows</button>
					<br><br>
					<button @click="toggleDisplayMode">
									{{ displayMode === 'pilgrims' ? 'Show as Rooms' : 'Show as Pilgrims' }}
					</button>
					<br>

					<!-- Container for the two tables -->
					<div class="table-container">
						<div class="header-table-container">
									<!-- Table for Date Headers -->
									<table class="header-table">
													<tr>
																	<th>ID</th>
																	<th v-for="date in dateRange" :key="date">{{ date.getDate() }}/{{ date.getMonth() + 1 }}</th>
													</tr>
													<tr v-for="(line, index) in roomLines" :key="index">
																	<td>{{ index + 1 }}</td>
																	<td v-for="date in dateRange" :key="date" @click="selectCell(index + 1, date)" :class="getCellClass(index + 1, date)">{{ getCellText(index + 1, date) }}</td>
													</tr>
									</table>
								</div>
									<!-- Table for Room Data -->
									<div class="data-table-container">

									<table class="data-table">
													<tr>
																	<th v-colspan="3">Room type</th>
																	<th v-colspan="3">Shifting Room type</th>
																	<th v-colspan="3">Rooms</th>
																	<th v-colspan="3">Confirmation</th>
													</tr>
													<tr v-for="(line, index) in roomLines" :key="index">
																	<!-- Room Type Dropdowns, Number of Rooms Input, and Confirmation Button -->
																	<td>
																					<!-- Main Room Type Dropdown -->
																					<select v-model="roomSettings[index + 1].mainRoomType">
																									<option value="double">Double (2 pilgrims)</option>
																									<option value="triple">Triple (3 pilgrims)</option>
																									<option value="quadruple">Quadruple (4 pilgrims)</option>
																					</select>
																	</td>
																	<td>
																					<!-- Shifting Room Type Dropdown -->
																					<select v-model="roomSettings[index + 1].shiftingRoomType" class="shifting-makkah-select">
																									<option value="double">Double (2 pilgrims)</option>
																									<option value="triple">Triple (3 pilgrims)</option>
																									<option value="quadruple">Quadruple (4 pilgrims)</option>
																					</select>
																	</td>
																	<td>
																					<!-- Number of Rooms Input -->
																					<input type="number" v-model.number="roomSettings[index + 1].numberOfRooms" min="1">
																	</td>
																	<td>
																					<!-- Confirmation Button -->
																					<button @click="toggleConfirmation(index + 1)"
																													@mouseover="hoverOnButton(index + 1, true)"
																													@mouseleave="hoverOnButton(index + 1, false)"
																													:class="{
																																	'confirmed-button': confirmedRows[index + 1],
																																	'unconfirm-button': hoveredRows[index + 1] && confirmedRows[index + 1]
																													}">
																									{{ confirmedRows[index + 1] ? (hoveredRows[index + 1] ? 'Unconfirm' : 'Confirmed') : 'Confirm' }}
																					</button>
																	</td>
													</tr>
									</table>
									</div>
					</div>

					<br>
					<button @click="calculate">Calculate</button>
					<h3>Summary:</h3>
					<table class="summary-table">
      <tr>
        <th>ID</th>
        <th>Area</th>
        <th>From</th>
        <th>To</th>
        <th>Room Type</th>
        <th>Number of Rooms</th>
        <th>Pilgrims</th>
      </tr>
      <tr v-for="summary in summaryData" :key="summary.id">
        <td>{{ summary.room }}</td>
        <td>{{ summary.area }}</td>
        <td>{{ summary.from }}</td>
        <td>{{ summary.to }}</td>
        <td>{{ summary.roomType }}</td>
        <td>{{ summary.numberOfRooms }}</td>
        <td>{{ summary.pilgrims }}</td>
      </tr>
    </table>
	</div>
</template>

<script>
export default {
	name: 'HotelBooking',
	data() {
	return {
			selectedHotelLocation: '',
			roomLines: Array(10).fill(null),
			selectedDates: {},
			roomSettings: this.initializeRoomSettings(10),
			summaryData: [],
			confirmedRows: {},
			hoveredRows: {},
			displayMode: 'rooms', // Initial display mode
			startDate: new Date(2024, 4, 25), // May 25th, 2024
			endDate: new Date(2024, 6, 10),   // July 10th, 2024
			dateRange: [],
	};
},

created() {
			this.initializeDateRange();
	},

methods: {

	initializeDateRange() {
					let currentDate = new Date(this.startDate.getTime());
					this.dateRange = [];
					while (currentDate <= this.endDate) {
							this.dateRange.push(new Date(currentDate));
							currentDate.setDate(currentDate.getDate() + 1);
					}
			},

			clearUnconfirmedRows() {
	// Iterate through all rows
	for (let i = 1; i <= this.roomLines.length; i++) {
			// Check if row is not confirmed
			if (!this.confirmedRows[i]) {
					// Iterate through the dynamic date range and clear the selected dates for unconfirmed rows
					this.dateRange.forEach(date => {
							const dateString = this.formatDate(date);
							const key = `${i}-${dateString}`;
							if (this.selectedDates[key]) {
									delete this.selectedDates[key];
							}
					});
			}
	}
},


checkForArea(rowId, area) {
	return this.dateRange.some(date => {
			const dateString = this.formatDate(date);
			const key = `${rowId}-${dateString}`;
			return this.selectedDates[key] === area;
	});
},


isContinuousDates(rowId) {
	let firstSelectedDateIndex = null;
	let lastSelectedDateIndex = null;

	// Find the first and last selected dates
	this.dateRange.forEach((date, index) => {
			const dateString = this.formatDate(date);
			const key = `${rowId}-${dateString}`;
			if (this.selectedDates[key]) {
					if (firstSelectedDateIndex === null) firstSelectedDateIndex = index;
					lastSelectedDateIndex = index;
			}
	});

	if (firstSelectedDateIndex === null || lastSelectedDateIndex === null) {
			return false;
	}

	// Check if all dates between the first and last selected dates are also selected
	for (let index = firstSelectedDateIndex; index <= lastSelectedDateIndex; index++) {
			const date = this.dateRange[index];
			const dateString = this.formatDate(date);
			const key = `${rowId}-${dateString}`;
			if (!this.selectedDates[key]) {
					return false;
			}
	}

	return true;
},


			toggleConfirmation(rowId) {
					const hasMainMakkah = this.checkForArea(rowId, 'mainMakkah');
					const hasMadinah = this.checkForArea(rowId, 'madinah');
					const continuousDates = this.isContinuousDates(rowId);

					if (hasMainMakkah && hasMadinah && continuousDates) {
							this.confirmedRows = {
									...this.confirmedRows,
									[rowId]: !this.confirmedRows[rowId]
							};
					} else {
							alert('To confirm, ensure both Main Makkah and Madinah areas are selected and the dates are continuous.');
					}
			},





	hoverOnButton(rowId, isHovering) {
			this.hoveredRows = {
					...this.hoveredRows,
					[rowId]: isHovering
			};
	},

	confirmRow(rowId) {
	this.confirmedRows = {
			...this.confirmedRows,
			[rowId]: true
	};
	// Additional logic for a confirmed row
},



initializeRoomSettings(count) {
	let settings = {};
	for (let i = 1; i <= count; i++) {
			settings[i] = {
					mainRoomType: 'quadruple', // Set default room type as 'quadruple'
					shiftingRoomType: 'quadruple', // Set default shifting room type as 'quadruple'
					numberOfRooms: 1
			};
	}
	return settings;
},


addRoomLine() {
	const newLineIndex = this.roomLines.push(null) - 1;
	this.roomSettings[newLineIndex + 1] = {
			mainRoomType: 'quadruple', // Set default room type as 'quadruple'
			shiftingRoomType: 'quadruple', // Set default shifting room type as 'quadruple'
			numberOfRooms: 1
	};
},

selectCell(rowId, date) {
	if (this.confirmedRows[rowId]) {
			alert('Editing is disabled for confirmed rows.');
			return;
	}

	const dateString = this.formatDate(date);
	const key = `${rowId}-${dateString}`;

	if (!this.selectedHotelLocation) {
			alert('Please select a location first.');
			return;
	}

	if (this.selectedDates[key] && this.selectedDates[key] !== this.selectedHotelLocation) {
			alert('You cannot modify cells in a different area. Please select another area to edit.');
			return;
	}

	// Find any existing range for the same area on this row
	const existingRange = this.findExistingRange(rowId, this.selectedHotelLocation);

	if (this.selectedDates[key]) {
			// If deselecting a cell within a selected range, then the whole range should be deselected
			this.deselectDateRangeIncludingCell(rowId, dateString);
	} else if (existingRange) {
			// Expand or shrink the existing range to include the selected date
			this.adjustExistingRange(rowId, date, existingRange);
	} else {
			// No existing range, select the individual cell
			this.selectedDates[key] = this.selectedHotelLocation;
	}

	// Update to maintain reactivity
	this.selectedDates = { ...this.selectedDates };
},


findExistingRange(rowId, area) {
	let rangeStart = null;
	let rangeEnd = null;

	this.dateRange.forEach((date, index) => {
			const dateString = this.formatDate(date);
			const key = `${rowId}-${dateString}`;
			if (this.selectedDates[key] === area) {
					if (rangeStart === null) {
							rangeStart = index;
					}
					rangeEnd = index;
			}
	});

	if (rangeStart !== null) {
			return { start: rangeStart, end: rangeEnd };
	}
	return null;
},

adjustExistingRange(rowId, date, existingRange) {
	const dateIndex = this.dateRange.indexOf(date);
	const newStart = Math.min(dateIndex, existingRange.start);
	const newEnd = Math.max(dateIndex, existingRange.end);

	for (let i = newStart; i <= newEnd; i++) {
			const rangeDate = this.dateRange[i];
			const rangeDateString = this.formatDate(rangeDate);
			this.selectedDates[`${rowId}-${rangeDateString}`] = this.selectedHotelLocation;
	}
},



findClosestSelectedDate(rowId, targetDate) {
	let closestDateIndex = null;

	this.dateRange.forEach((date, index) => {
			const dateString = this.formatDate(date);
			const key = `${rowId}-${dateString}`;

			if (this.selectedDates[key]) {
					if (closestDateIndex === null || Math.abs(index - this.dateRange.indexOf(targetDate)) < Math.abs(closestDateIndex - this.dateRange.indexOf(targetDate))) {
							closestDateIndex = index;
					}
			}
	});

	return closestDateIndex;
},

deselectDateRangeIncludingCell(rowId, dateString) {
	let startIndex = this.dateRange.findIndex(d => this.formatDate(d) === dateString);
	let endIndex = startIndex;

	for (let i = startIndex - 1; i >= 0; i--) {
			const checkDateString = this.formatDate(this.dateRange[i]);
			if (this.selectedDates[`${rowId}-${checkDateString}`] === this.selectedHotelLocation) {
					startIndex = i;
			} else {
					break;
			}
	}

	for (let i = startIndex + 1; i < this.dateRange.length; i++) {
			const checkDateString = this.formatDate(this.dateRange[i]);
			if (this.selectedDates[`${rowId}-${checkDateString}`] === this.selectedHotelLocation) {
					endIndex = i;
			} else {
					break;
			}
	}

	for (let i = startIndex; i <= endIndex; i++) {
			const rangeDateString = this.formatDate(this.dateRange[i]);
			delete this.selectedDates[`${rowId}-${rangeDateString}`];
	}

	this.selectedDates = { ...this.selectedDates };
},

toggleDisplayMode() {
					this.displayMode = this.displayMode === 'rooms' ? 'pilgrims' : 'rooms';
			},

			getCellText(rowId, date) {
	const dateString = this.formatDate(date);
	const key = `${rowId}-${dateString}`;
	const location = this.selectedDates[key];

	if (location) {
			const roomSetting = this.roomSettings[rowId];

			if (location === 'shiftingMakkah') {
					const mainMakkahPilgrims = this.calculatePilgrims('mainMakkah', rowId);
					const shiftingRoomType = roomSetting.shiftingRoomType || 'quadruple';
					const roomCapacity = this.getRoomCapacity(shiftingRoomType);
					const numberOfRooms = Math.ceil(mainMakkahPilgrims / roomCapacity);

					return this.displayMode === 'rooms'
							? (numberOfRooms > 0 ? numberOfRooms.toString() : '-')
							: mainMakkahPilgrims.toString();
			} else {
					const roomCapacity = this.getRoomCapacity(roomSetting.mainRoomType);
					const numberOfRooms = roomSetting.numberOfRooms || 0;
					const pilgrims = roomCapacity * numberOfRooms;

					return this.displayMode === 'rooms'
							? numberOfRooms.toString()
							: pilgrims.toString();
			}
	}
	return '-';
},


getCellClass(rowId, date) {
	const dateString = this.formatDate(date);
	const key = `${rowId}-${dateString}`;
	const location = this.selectedDates[key];
	return location ? location : '';
},


calculate() {
	this.summaryData = [];
	let map = new Map();

	for (const [key, area] of Object.entries(this.selectedDates)) {
			const [rowId, dateString] = key.split('-');
			if (!this.confirmedRows[rowId]) {
					continue; // Skip unconfirmed rows
			}
			// Correctly parse the dateString into a Date object
			const [day, month] = dateString.split('/');
			const date = new Date(2024, month - 1, day); // Year is fixed as 2024
			const mapKey = `${area}-${rowId}`;

			if (!map.has(mapKey)) {
					map.set(mapKey, []);
			}
			map.get(mapKey).push(date);
	}

	map.forEach((dates, key) => {
			const [area, rowId] = key.split('-');
			this.pushSummaryData(area, rowId, dates);
	});
},

pushSummaryData(area, rowId, dates) {
	dates.sort((a, b) => a - b);
	let startDate = dates[0];
	let endDate = dates[dates.length - 1];

	const formattedStartDate = this.formatDate(startDate);
	const formattedEndDate = this.formatDate(endDate);

	if (area === 'mainMakkah' || area === 'madinah') {
			const roomType = this.roomSettings[rowId]?.mainRoomType || '';
			const numberOfRooms = this.roomSettings[rowId]?.numberOfRooms || 1;
			const pilgrims = this.getRoomCapacity(roomType) * numberOfRooms;

			this.summaryData.push({
					id: this.summaryData.length + 1,
					area,
					from: formattedStartDate,
					to: formattedEndDate,
					room: rowId,
					roomType,
					numberOfRooms,
					pilgrims,
			});
	} else if (area === 'shiftingMakkah') {
			const mainMakkahPilgrims = this.calculatePilgrims('mainMakkah', rowId);
			let shiftingRoomType = this.roomSettings[rowId]?.shiftingRoomType || '';
			const roomCapacity = this.getRoomCapacity(shiftingRoomType);

			if (mainMakkahPilgrims % roomCapacity !== 0) {
					// Adjusting Shifting Makkah room type to match Main Makkah
					shiftingRoomType = this.roomSettings[rowId].mainRoomType;
					this.roomSettings[rowId].shiftingRoomType = shiftingRoomType;
			}

			const numberOfRooms = Math.ceil(mainMakkahPilgrims / this.getRoomCapacity(shiftingRoomType));

			this.summaryData.push({
					id: this.summaryData.length + 1,
					area: 'Shifting Makkah',
					from: formattedStartDate,
					to: formattedEndDate,
					room: rowId,
					roomType: shiftingRoomType,
					numberOfRooms,
					pilgrims: mainMakkahPilgrims,
			});
	}
},






calculatePilgrims(area, rowId) {
	if (area === 'mainMakkah') {
			const roomType = this.roomSettings[rowId]?.mainRoomType || '';
			const numberOfRooms = this.roomSettings[rowId]?.numberOfRooms || 1;
			return this.getRoomCapacity(roomType) * numberOfRooms;
	}
	return 0;
},




formatDate(date) {
	let day = date.getDate().toString().padStart(2, '0');
	let month = (date.getMonth() + 1).toString().padStart(2, '0'); // JavaScript months are 0-indexed
	return `${day}/${month}`;
},




			getRoomCapacity(type) {
					switch (type) {
							case 'double': return 2;
							case 'triple': return 3;
							case 'quadruple': return 4;
							default: return 0;
					}
			},
	},

	


};
</script>

// 
<style scoped>
/* Common table styles */
.table-container {
    display: flex;
    /* Remove overflow-x: auto; from the table container */
				width: 100%;
}

.header-table-container {
    overflow-x: auto; /* Add horizontal scroll to the header table container */
    margin-bottom: 10px; /* Add some space below the header table */
}

.data-table,
.summary-table {
    width: 30%;
}

.header-table,
 {
    width: 70%;
}

/* Ensure all td elements have the same height */
.header-table td,
.data-table td {
    height: 32px; /* Adjusted height value */
}

.header-table th,
.data-table th {
    height: 32px; /* Adjusted height value */
}

/* Enforce fixed width for table cells */
.header-table {
    table-layout: fixed;
}

.header-table th,
.data-table th {
   width: 40px;
}

.header-table th,
.data-table th,
.summary-table th,
.header-table td,
.data-table td,
.summary-table td {
    border: 1px solid black;
    text-align: center;
}


.header-table th,
.data-table th,
.summary-table th {
    background-color: #f2f2f2;
}



.header-table td {
    background-color: white;
    cursor: default;
}

/* Additional styles */
.mainMakkah { background-color: #ffcccc; }
.shiftingMakkah { background-color: #ccccff; }
.madinah { background-color: #ccffcc; }

.confirmed-button {
    background-color: green;
    color: white;
}

.unconfirm-button {
    background-color: red;
    color: white;
}

.shifting-makkah-select {
    background-color: lightblue;
}

input[type="number"] {
    width: 50px; /* Adjusted for better visibility */
}

/* Cell coloring based on location */
.header-table td.mainMakkah { background-color: #ffcccc; }
.header-table td.shiftingMakkah { background-color: #ccccff; }
.header-table td.madinah { background-color: #ccffcc; }

.data-table td.mainMakkah { background-color: #ffcccc; }
.data-table td.shiftingMakkah { background-color: #ccccff; }
.data-table td.madinah { background-color: #ccffcc; }
</style>




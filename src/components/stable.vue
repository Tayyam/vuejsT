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
					<button @click="getCheckedRowsData">Get Checked Data</button>
					<br><br>
					<button @click="resetLocalStorage">Reset Local Storage</button>
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
																	<th>room</th>
																	<th v-for="date in dateRange" :key="date" :class="{ 'holy-period': isHolyPeriod(date) }">
            {{ date.getDate() }}/{{ date.getMonth() + 1 }}
        </th>

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
																	<th v-colspan="3">Check</th>
																	
													</tr>
													<tr v-for="(line, index) in roomLines" :key="index">
																	<!-- Room Type Dropdowns, Number of Rooms Input, and Confirmation Button -->
																	<td>
																					<!-- Main Room Type Dropdown -->
																					<select v-model="roomSettings[index + 1].mainRoomType" :disabled="confirmedRows[index + 1]">
            <option value="double">Double (2 pilgrims)</option>
            <option value="triple">Triple (3 pilgrims)</option>
            <option value="quadruple">Quadruple (4 pilgrims)</option>
        </select>
																	</td>
																	<td>
																					<!-- Shifting Room Type Dropdown -->
																					<select v-model="roomSettings[index + 1].shiftingRoomType" class="shifting-makkah-select" :id="'shiftingRoomType-' + (index + 1)" :disabled="confirmedRows[index + 1]">
            <option value="double">Double (2 pilgrims)</option>
            <option value="triple">Triple (3 pilgrims)</option>
            <option value="quadruple">Quadruple (4 pilgrims)</option>
        </select>

																	</td>
																	<td>
																					<!-- Number of Rooms Input -->
																					<input type="number" v-model.number="roomSettings[index + 1].numberOfRooms" min="1" :disabled="confirmedRows[index + 1]">
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
																	<td>
    <input type="checkbox" 
           @change="handleCheckboxChange(index + 1, $event)" 
           :disabled="!confirmedRows[index + 1]">
</td>
													</tr>
									</table>
									</div>
					</div>

					<br>
					<!-- <button @click="calculate">Calculate</button> -->
					<h3>Summary:</h3>
					<table class="summary-table">
      <tr>
        <th>room</th>
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
            holyPeriodStart: new Date(2024, 5, 15), // Holy period start date: June 15th, 2024
            holyPeriodEnd: new Date(2024, 5, 19),   // Holy period end date: June 19th, 2024
            dateRange: [],
            checkedRows: {},
        };
    },

    watch: {
        roomSettings: {
            handler: 'adjustShiftingMakkahRoomType',
            deep: true
        }
    },

    created() {
        this.initializeDateRange();
        this.loadSummaryData();
    },

    methods: {

        handleCheckboxChange(rowIndex, event) {
            this.checkedRows[rowIndex] = event.target.checked;
        },

        getCheckedRowsData() {
            let combinedData = [];

            // Iterate over each row
            for (let rowIndex = 1; rowIndex <= this.roomLines.length; rowIndex++) {
                // Check if the row is checked
                if (this.checkedRows[rowIndex]) {
                    let rowData = [];

                    // Process each selected date for the checked row
                    for (const [key, areas] of Object.entries(this.selectedDates)) {
                        const [currentRowId, dateString] = key.split('-');
                        if (parseInt(currentRowId) === rowIndex) {
                            areas.forEach(area => {
                                const date = this.parseDateFromString(dateString);
                                this.addToRowData(rowData, rowIndex, area, date);
                            });
                        }
                    }

                    // Combine the processed row data into a single array
                    if (rowData.length > 0) {
                        combinedData.push(...rowData);
                    }
                }
            }

            console.log("Combined Data for Checked Rows:", combinedData);
        },

        logRowContents(rowIndex) {
            let rowData = [];

            // Process each selected date for the row
            for (const [key, areas] of Object.entries(this.selectedDates)) {
                const [currentRowId, dateString] = key.split('-');
                if (parseInt(currentRowId) === rowIndex) {
                    areas.forEach(area => {
                        const date = this.parseDateFromString(dateString);
                        this.addToRowData(rowData, rowIndex, area, date);
                    });
                }
            }

            // Log the row data in a structured format
            console.log(rowData);
        },

        parseDateFromString(dateString) {
            // Parse the dateString into a Date object
            const [day, month] = dateString.split('/');
            return new Date(2024, month - 1, day); // Example year, adjust as necessary
        },

        addToRowData(rowData, rowId, area, date) {
            // Find existing entry or create a new one
            let entry = rowData.find(entry => entry.area === area);
            if (!entry) {
                entry = {
                    area: area,
                    from: this.formatDate(date),
                    to: this.formatDate(date),
                    room: rowId.toString(),
                    roomType: this.roomSettings[rowId]?.mainRoomType || 'unknown', // Adjust according to your data structure
                    numberOfRooms: this.roomSettings[rowId]?.numberOfRooms || 1
                };
                rowData.push(entry);
            } else {
                // Update 'to' date if the new date is later
                if (date > this.parseDateFromString(entry.to)) {
                    entry.to = this.formatDate(date);
                }
            }
        },

        resetLocalStorage() {
            localStorage.clear();
            alert('Local storage has been reset.');
        },

        checkForAreaDuringHolyPeriod(rowId, area) {
            // Iterate through the holy period dates
            for (let date = new Date(this.holyPeriodStart); date <= this.holyPeriodEnd; date.setDate(date.getDate() + 1)) {
                const dateString = this.formatDate(date);
                const key = `${rowId}-${dateString}`;
                // Check if the area is selected during the holy period
                if (this.selectedDates[key] && this.selectedDates[key].includes(area)) {
                    return true; // Area is selected during the holy period
                }
            }
            return false; // Area is not selected during the holy period
        },

        adjustShiftingMakkahRoomType() {
            Object.entries(this.roomSettings).forEach(([lineId, settings]) => {
                const mainMakkahPilgrims = this.calculatePilgrims('mainMakkah', parseInt(lineId));
                const roomCapacity = this.getRoomCapacity(settings.shiftingRoomType);

                // Check if the number of pilgrims for Shifting Makkah is not an integer multiple of the room capacity
                if (mainMakkahPilgrims % roomCapacity !== 0) {
                    // Alert the user before making changes
                    alert(`Adjusting Shifting Makkah room type for line ${lineId} to match Main Makkah room type due to pilgrim count.`);

                    // Adjust the Shifting Makkah room type to match the Main Makkah room type
                    this.roomSettings[lineId].shiftingRoomType = this.roomSettings[lineId].mainRoomType;

                    // Change the input selection immediately
                    document.getElementById(`shiftingRoomType-${lineId}`).value = this.roomSettings[lineId].shiftingRoomType;
                }
            });
        },

        isHolyPeriod(date) {
            return date >= this.holyPeriodStart && date <= this.holyPeriodEnd;
        },

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
                return this.selectedDates[key] && this.selectedDates[key].includes(area);
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
            // Checks for area selection and date continuity
            const hasMainMakkah = this.checkForArea(rowId, 'mainMakkah');
            const hasShiftingMakkah = this.checkForArea(rowId, 'shiftingMakkah');
            const hasMadinah = this.checkForArea(rowId, 'madinah');
            const continuousDates = this.isContinuousDates(rowId);
            const isHolyPeriodIncluded = this.isHolyPeriodIncluded(rowId);
            const hasMadinahDuringHolyPeriod = this.checkForAreaDuringHolyPeriod(rowId, 'madinah');

            // Define area requirements
            const areaRequirements = {
                mainMakkah: { min: 3, max: 15 },
                shiftingMakkah: { min: 3, max: 15 },
                madinah: { min: 2, max: 15 },
                overall: { min: 13, max: 25 },
            };

            // Calculate selected days for each area
            const selectedDays = this.calculateSelectedDaysForAreas(rowId);

            let alertMessages = [];
            Object.entries(areaRequirements).forEach(([area, { min, max }]) => {
                if (area === 'overall') {
                    if (selectedDays.total < min || selectedDays.total > max) {
                        alertMessages.push(`Total period must be between ${min} and ${max} days. Currently selected: ${selectedDays.total} days.`);
                    }
                } else if (selectedDays[area] > 0 && (selectedDays[area] < min || selectedDays[area] > max)) {
                    alertMessages.push(`${area.charAt(0).toUpperCase() + area.slice(1)} period must be between ${min} and ${max} days. Currently selected: ${selectedDays[area]} days.`);
                }
            });

            // Check for intersections
            const intersections = this.countIntersections(rowId);
            let intersectionRequirementMet = false;

            if (hasShiftingMakkah && intersections === 2) {
                intersectionRequirementMet = true;
            } else if (!hasShiftingMakkah && intersections === 1) {
                intersectionRequirementMet = true;
            }

            if (!intersectionRequirementMet) {
                alertMessages.push("Intersection requirements not met.");
            }

            // Add messages for initial checks if needed
            if (!hasMainMakkah || !hasMadinah || !continuousDates || !isHolyPeriodIncluded || hasMadinahDuringHolyPeriod || !intersectionRequirementMet) {
                if (!hasMainMakkah) alertMessages.push("Main Makkah area is not selected.");
                if (!hasMadinah) alertMessages.push("Madinah area is not selected.");
                if (!continuousDates) alertMessages.push("Selected dates are not continuous.");
                if (!isHolyPeriodIncluded) alertMessages.push("Holy period is not fully included.");
                if (hasMadinahDuringHolyPeriod) alertMessages.push("Madinah area is selected during the holy period.");
            }

            // Toggle confirmation and calculate summary if no issues found
            if (alertMessages.length === 0) {
                this.confirmedRows[rowId] = !this.confirmedRows[rowId];
                // Call the calculate method to update summary data
                this.calculate();
            } else {
                // Display all collected alert messages
                alert(alertMessages.join("\n"));
            }
        },

        countIntersections(rowId) {
            let intersectionCount = 0;

            this.dateRange.forEach(date => {
                const dateString = this.formatDate(date);
                const key = `${rowId}-${dateString}`;
                const locations = this.selectedDates[key];

                if (locations && locations.length > 1) {
                    intersectionCount++;
                }
            });

            return intersectionCount;
        },

        calculateSelectedDaysForAreas(rowId) {
            const areaCounts = {
                mainMakkah: 0,
                shiftingMakkah: 0,
                madinah: 0,
                total: 0
            };

            // Iterate through all selected dates
            Object.entries(this.selectedDates).forEach(([key, value]) => {
                // Check if the current key belongs to the given rowId
                if (key.startsWith(rowId + '-')) {
                    // Increment total days count
                    areaCounts.total += 1;

                    // Increment specific area counts based on the value (which is an array of selected areas)
                    value.forEach(area => {
                        if (area === 'mainMakkah') {
                            areaCounts.mainMakkah += 1;
                        } else if (area === 'shiftingMakkah') {
                            areaCounts.shiftingMakkah += 1;
                        } else if (area === 'madinah') {
                            areaCounts.madinah += 1;
                        }
                    });
                }
            });

            return areaCounts;
        },

        isHolyPeriodIncluded(rowId) {

            for (let date = new Date(this.holyPeriodStart); date <= this.holyPeriodEnd; date.setDate(date.getDate() + 1)) {
                const dateString = this.formatDate(date);
                const key = `${rowId}-${dateString}`;
                if (!this.selectedDates[key]) {
                    return false; // Holy period is not fully included
                }
            }

            return true; // Holy period is fully included
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

            // Toggle the selection
            if (!this.selectedDates[key]) {
                // First, check if adding to this cell violates the new rule
                const isAtStartOrEnd = this.isAtStartOrEndOfOtherArea(rowId, date);
                if (isAtStartOrEnd || !this.anySelectionExists(key)) {
                    // Either at the start/end of an existing area or no conflict, proceed to select
                    this.selectedDates[key] = [this.selectedHotelLocation];
                } else {
                    // Violates the rule, do not allow selection
                    alert('You can only select the start or end date of an existing selection for a different area.');
                    return;
                }
            } else {
                const locationIndex = this.selectedDates[key].indexOf(this.selectedHotelLocation);
                if (locationIndex === -1) {
                    // Check rule before adding a different area to the selection
                    const isAtStartOrEnd = this.isAtStartOrEndOfOtherArea(rowId, date);
                    if (isAtStartOrEnd) {
                        this.selectedDates[key].push(this.selectedHotelLocation);
                    } else {
                        alert('You can only select the start or end date of an existing selection for a different area.');
                        return;
                    }
                } else {
                    // Deselecting an existing selection
                    this.selectedDates[key].splice(locationIndex, 1);
                    if (this.selectedDates[key].length === 0) {
                        delete this.selectedDates[key];
                    }
                    // Check for in-between dates to deselect if necessary
                    this.deselectInBetweenDatesIfNecessary(rowId, this.selectedHotelLocation);
                    this.selectedDates = { ...this.selectedDates };
                    return;
                }
            }

            const existingRange = this.findExistingRange(rowId, this.selectedHotelLocation);
            if (existingRange) {
                this.adjustExistingRange(rowId, date, existingRange);
            }

            this.selectedDates = { ...this.selectedDates };
        },

        anySelectionExists(key) {
            return this.selectedDates[key] && this.selectedDates[key].length > 0;
        },

        isAtStartOrEndOfOtherArea(rowId, date) {
            let atStartOrEnd = false;
            this.dateRange.forEach((otherDate) => {
                const otherDateString = this.formatDate(otherDate);
                const otherKey = `${rowId}-${otherDateString}`;
                if (this.selectedDates[otherKey] && this.selectedDates[otherKey].length > 0) {
                    const otherAreas = this.selectedDates[otherKey].filter(area => area !== this.selectedHotelLocation);
                    otherAreas.forEach((otherArea) => {
                        const range = this.findExistingRange(rowId, otherArea);
                        if (range) {
                            const dateIndex = this.dateRange.findIndex(d => this.formatDate(d) === this.formatDate(date));
                            if (dateIndex === range.start || dateIndex === range.end) {
                                atStartOrEnd = true;
                            }
                        }
                    });
                }
            });
            return atStartOrEnd;
        },

        deselectInBetweenDatesIfNecessary(rowId, area) {
            this.dateRange.forEach(date => {
                const dateString = this.formatDate(date);
                const key = `${rowId}-${dateString}`;
                if (this.selectedDates[key] && this.selectedDates[key].includes(area)) {
                    this.selectedDates[key] = this.selectedDates[key].filter(location => location !== area);
                    if (this.selectedDates[key].length === 0) {
                        delete this.selectedDates[key];
                    }
                }
            });
        },










        findExistingRange(rowId, area) {
            let rangeStart = null;
            let rangeEnd = null;

            this.dateRange.forEach((date, index) => {
                const dateString = this.formatDate(date);
                const key = `${rowId}-${dateString}`;
                if (this.selectedDates[key] && this.selectedDates[key].includes(area)) {
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
                const key = `${rowId}-${rangeDateString}`;
                if (!this.selectedDates[key]) {
                    this.selectedDates[key] = [this.selectedHotelLocation];
                } else if (!this.selectedDates[key].includes(this.selectedHotelLocation)) {
                    this.selectedDates[key].push(this.selectedHotelLocation);
                }
            }
        },

        deselectDateRangeIncludingCell(rowId, dateString) {
            let startIndex = this.dateRange.findIndex(d => this.formatDate(d) === dateString);
            let endIndex = startIndex;

            for (let i = startIndex - 1; i >= 0; i--) {
                const checkDateString = this.formatDate(this.dateRange[i]);
                const key = `${rowId}-${checkDateString}`;
                if (this.selectedDates[key] && this.selectedDates[key].includes(this.selectedHotelLocation)) {
                    startIndex = i;
                } else {
                    break;
                }
            }

            for (let i = startIndex + 1; i < this.dateRange.length; i++) {
                const checkDateString = this.formatDate(this.dateRange[i]);
                const key = `${rowId}-${checkDateString}`;
                if (this.selectedDates[key] && this.selectedDates[key].includes(this.selectedHotelLocation)) {
                    endIndex = i;
                } else {
                    break;
                }
            }

            for (let i = startIndex; i <= endIndex; i++) {
                const rangeDateString = this.formatDate(this.dateRange[i]);
                const key = `${rowId}-${rangeDateString}`;
                const locationIndex = this.selectedDates[key].indexOf(this.selectedHotelLocation);
                if (locationIndex > -1) {
                    this.selectedDates[key].splice(locationIndex, 1);
                    if (this.selectedDates[key].length === 0) {
                        delete this.selectedDates[key];
                    }
                }
            }

            this.selectedDates = { ...this.selectedDates };
        },
        findClosestSelectedDate(rowId, targetDate) {
            let closestDateIndex = null;
            let minDistance = Infinity;

            this.dateRange.forEach((date, index) => {
                const dateString = this.formatDate(date);
                const key = `${rowId}-${dateString}`;

                if (this.selectedDates[key] && this.selectedDates[key].includes(this.selectedHotelLocation)) {
                    const distance = Math.abs(index - this.dateRange.indexOf(targetDate));
                    if (distance < minDistance) {
                        closestDateIndex = index;
                        minDistance = distance;
                    }
                }
            });

            return closestDateIndex !== null ? this.dateRange[closestDateIndex] : null;
        },






        toggleDisplayMode() {
            this.displayMode = this.displayMode === 'rooms' ? 'pilgrims' : 'rooms';
        },

        getCellText(rowId, date) {
            const dateString = this.formatDate(date);
            const key = `${rowId}-${dateString}`;
            const locations = this.selectedDates[key] || []; // Fallback to an empty array

            let textRepresentation = '-'; // Initialize textRepresentation outside the map

            if (locations.length) {
                textRepresentation = locations.map(location => {
                    const roomSetting = this.roomSettings[rowId] || {
                        mainRoomType: 'quadruple',
                        shiftingRoomType: 'quadruple',
                        numberOfRooms: 1
                    };

                    if (location === 'shiftingMakkah') {
                        const mainMakkahPilgrims = this.calculatePilgrims('mainMakkah', rowId);
                        const shiftingRoomType = roomSetting.shiftingRoomType;
                        const roomCapacity = this.getRoomCapacity(shiftingRoomType);
                        const numberOfRooms = Math.ceil(mainMakkahPilgrims / roomCapacity);

                        return this.displayMode === 'rooms' ?
                            `${numberOfRooms}` :
                            `${mainMakkahPilgrims}`;
                    } else {
                        const roomType = roomSetting.mainRoomType;
                        const roomCapacity = this.getRoomCapacity(roomType);
                        const numberOfRooms = roomSetting.numberOfRooms;
                        const pilgrims = roomCapacity * numberOfRooms;

                        return this.displayMode === 'rooms' ?
                            `${numberOfRooms}` :
                            `${pilgrims}`;
                    }
                }).join(', '); // Join the results from map
            }

            return textRepresentation;
        },






        getCellClass(rowId, date) {
            const dateString = this.formatDate(date);
            const key = `${rowId}-${dateString}`;
            const locations = this.selectedDates[key] || [];

            if (locations.length > 1) {
                return locations.join('-') + '-intersection';
            } else if (locations.length === 1) {
                return locations[0];
            }

            return '';
        },






        calculate() {
            this.summaryData = [];
            let map = new Map();

            // Iterate through selectedDates and construct the map with aggregated data
            for (const [key, areas] of Object.entries(this.selectedDates)) {
                const [rowId, dateString] = key.split('-');
                if (!this.confirmedRows[rowId]) {
                    continue; // Skip unconfirmed rows
                }

                const [day, month] = dateString.split('/'); // Split dateString into day and month
                const date = new Date(2024, month - 1, day); // Parse the date string directly
                areas.forEach(area => {
                    const mapKey = `${area}-${rowId}`;
                    if (!map.has(mapKey)) {
                        map.set(mapKey, []);
                    }
                    map.get(mapKey).push(date);
                });
            }

            // Process each area-rowId pair in the map
            map.forEach((dates, key) => {
                this.processAreaDates(key, dates);
            });

            // After calculating, save the summaryData to local storage
            this.saveSummaryData();
        },

        saveSummaryData() {
            localStorage.setItem('summaryData', JSON.stringify(this.summaryData));
        },

        loadSummaryData() {
            const storedData = localStorage.getItem('summaryData');
            if (storedData) {
                this.summaryData = JSON.parse(storedData);
                this.reconstructStateFromSummary();
            }
        },

        reconstructStateFromSummary() {
        // Reset existing state
        this.roomLines = [];
        this.selectedDates = {};
        this.roomSettings = {};
        this.confirmedRows = {};

        // Create a map to track original room numbers to new sequential numbers
        const roomNumberMap = new Map();
        let nextRoomNumber = 1;

        this.summaryData.forEach(summary => {
            const originalRoomNumber = parseInt(summary.room);

            // Check if this original room number already has a mapped new number
            if (!roomNumberMap.has(originalRoomNumber)) {
                roomNumberMap.set(originalRoomNumber, nextRoomNumber++);
            }

            const newRoomNumber = roomNumberMap.get(originalRoomNumber);

            // Adjust summary to reflect new room number
            summary.room = newRoomNumber.toString();

            // Update other data structures as required
            if (!this.roomLines[newRoomNumber - 1]) {
                this.roomLines[newRoomNumber - 1] = null;
                this.roomSettings[newRoomNumber] = {
                    mainRoomType: summary.roomType,
                    shiftingRoomType: summary.roomType, // Assuming the same room type for shifting
                    numberOfRooms: summary.numberOfRooms
                };
                this.populateSelectedDatesFromSummary(summary, newRoomNumber);
                this.confirmedRows[newRoomNumber] = true;
            }
        });

        // Force a UI update
        this.selectedDates = { ...this.selectedDates };
        this.roomSettings = { ...this.roomSettings };
        this.confirmedRows = { ...this.confirmedRows };
    },

    populateSelectedDatesFromSummary(summary, lineIndex) {
        let current = new Date(2024, parseInt(summary.from.split('/')[1]) - 1, parseInt(summary.from.split('/')[0]));
        const end = new Date(2024, parseInt(summary.to.split('/')[1]) - 1, parseInt(summary.to.split('/')[0]));
        while (current <= end) {
            const key = `${lineIndex}-${this.formatDate(current)}`;
            const area = summary.area === 'Shifting Makkah' ? 'shiftingMakkah' : summary.area;
            if (!this.selectedDates[key]) {
                this.selectedDates[key] = [area];
            } else if (!this.selectedDates[key].includes(area)) {
                this.selectedDates[key].push(area);
            }
            current.setDate(current.getDate() + 1);
        }
    },





        // Helper method to get all dates in a range
        getDatesInRange(startDate, endDate) {
            const start = new Date(startDate);
            const end = new Date(endDate);
            const dates = [];
            for (let dt = new Date(start); dt <= end; dt.setDate(dt.getDate() + 1)) {
                dates.push(new Date(dt));
            }
            return dates;
        },


        processAreaDates(mapKey, dates) {
            const [area, rowId] = mapKey.split('-');
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
                    area,
                    from: formattedStartDate,
                    to: formattedEndDate,
                    room: rowId,
                    roomType,
                    numberOfRooms,
                    pilgrims
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

.header-table, {
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

/* Intersection styles */
.mainMakkah-shiftingMakkah-intersection,
.shiftingMakkah-mainMakkah-intersection {
    background: linear-gradient(to bottom, #ffcccc 50%, #ccccff 50%);
}

.mainMakkah-madinah-intersection,
.madinah-mainMakkah-intersection {
    background: linear-gradient(to bottom, #ffcccc 50%, #ccffcc 50%);
}

.shiftingMakkah-madinah-intersection,
.madinah-shiftingMakkah-intersection {
    background: linear-gradient(to bottom, #ccccff 50%, #ccffcc 50%);
}

/* Style for the holy period header */
.header-table .holy-period {
    background-color: gold;
}

</style>
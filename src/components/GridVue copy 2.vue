
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
<br>

    <br>
    <table>
      <tr>
        <th>ID</th>
        <th v-for="day in 30" :key="day">{{ day }}/6</th>
        <th v-colspan="3">Room type</th>
        <th v-colspan="3">Shifting Room type</th>
        <th v-colspan="3">Rooms</th>
        <th v-colspan="3">Confirmation</th>
      </tr>
      <tr v-for="(line, index) in roomLines" :key="index">
        <td>{{ index + 1 }}</td>
        <td v-for="day in 30" :key="day" @click="selectCell(index + 1, day)" :class="getCellClass(index + 1, day)">{{ getCellText(index + 1, day) }}</td>
        <!-- Main Room Type Dropdown -->
        <td>
          <select v-model="roomSettings[index + 1].mainRoomType">
            <option value="double">Double (2 pilgrims)</option>
            <option value="triple">Triple (3 pilgrims)</option>
            <option value="quadruple">Quadruple (4 pilgrims)</option>
            
          </select>
        </td>
        <!-- Shifting Room Type Dropdown -->
        <td>
          <select v-model="roomSettings[index + 1].shiftingRoomType" class="shifting-makkah-select">
            <option value="double">Double (2 pilgrims)</option>
            <option value="triple">Triple (3 pilgrims)</option>
            <option value="quadruple">Quadruple (4 pilgrims)</option>
          </select>
        </td>
        <!-- Number of Rooms Input -->
        <td>
          <input type="number" v-model.number="roomSettings[index + 1].numberOfRooms" min="1">
        </td>
        <td>
          <button 
  @click="toggleConfirmation(index + 1)"
  @mouseover="hoverOnButton(index + 1, true)"
  @mouseleave="hoverOnButton(index + 1, false)"
  :class="{
    'confirmed-button': confirmedRows[index + 1],
    'unconfirm-button': hoveredRows[index + 1] && confirmedRows[index + 1]
  }"
>
  {{ confirmedRows[index + 1] ? (hoveredRows[index + 1] ? 'Unconfirm' : 'Confirmed') : 'Confirm' }}
</button>

  </td>
      </tr>
    </table>
    <br>
    <button @click="calculate">Calculate</button>
    <h3>Summary:</h3>
    <table>
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
  };
},

methods: {

  clearUnconfirmedRows() {
    // Iterate through all rows
    for (let i = 1; i <= this.roomLines.length; i++) {
      // Check if row is not confirmed
      if (!this.confirmedRows[i]) {
        // Iterate through all days and clear the selected dates for unconfirmed rows
        for (let day = 1; day <= 30; day++) {
          const key = `${i}-${day}`;
          if (this.selectedDates[key]) {
            delete this.selectedDates[key];
          }
        }
      }
    }
  },

  isContinuousDates(rowId) {
  let firstSelectedDay = null;
  let lastSelectedDay = null;

  // Find the first and last selected days
  for (let day = 1; day <= 30; day++) {
    const key = `${rowId}-${day}`;
    if (this.selectedDates[key]) {
      if (firstSelectedDay === null) firstSelectedDay = day;
      lastSelectedDay = day;
    }
  }

  // Check for continuous selection between the first and last selected days
  for (let day = firstSelectedDay; day <= lastSelectedDay; day++) {
    const key = `${rowId}-${day}`;
    if (!this.selectedDates[key]) {
      // Found a gap within the selected range
      return false;
    }
  }

  // No gaps found within the range
  return true;
},

toggleConfirmation(rowId) {
  // Check if both Main Makkah and Madinah are selected
  const hasMainMakkah = this.checkForArea(rowId, 'mainMakkah');
  const hasMadinah = this.checkForArea(rowId, 'madinah');

  if (hasMainMakkah && hasMadinah) {
    this.confirmedRows = {
      ...this.confirmedRows,
      [rowId]: !this.confirmedRows[rowId]
    };
  } else {
    alert('To confirm, please ensure both Main Makkah and Madinah areas are selected.');
  }
},

checkForArea(rowId, area) {
  for (let day = 1; day <= 30; day++) {
    const key = `${rowId}-${day}`;
    if (this.selectedDates[key] === area) {
      return true;
    }
  }
  return false;
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

    selectCell(rowId, day) {
   // Check if the row is confirmed. If so, prevent editing.
   if (this.confirmedRows[rowId]) {
    alert('Editing is disabled for confirmed rows.');
    return;
  }

  const key = `${rowId}-${day}`;

  // Check if a location is selected
  if (!this.selectedHotelLocation) {
    alert('Please select a location first.');
    return;
  }

  // Check if the cell belongs to the same area as the selected area
  if (this.selectedDates[key] && this.selectedDates[key] !== this.selectedHotelLocation) {
    alert('You cannot modify cells in a different area. Please select another area to edit.');
    return;
  }

  // Set the flag to true to indicate that the user is currently selecting cells
  this.isSelectingCell = true;

  // If the cell is already selected
  if (this.selectedDates[key]) {
    let rangeStart = day;
    let rangeEnd = day;

    // Find the start of the range
    for (let d = day - 1; d >= 1; d--) {
      if (this.selectedDates[`${rowId}-${d}`] === this.selectedHotelLocation) {
        rangeStart = d;
      } else {
        break;
      }
    }

    // Find the end of the range
    for (let d = day + 1; d <= 30; d++) {
      if (this.selectedDates[`${rowId}-${d}`] === this.selectedHotelLocation) {
        rangeEnd = d;
      } else {
        break;
      }
    }

    // If the deselected cell is within a range, remove the whole range
    if (rangeStart !== day || rangeEnd !== day) {
      for (let d = rangeStart; d <= rangeEnd; d++) {
        delete this.selectedDates[`${rowId}-${d}`];
      }
    } else {
      delete this.selectedDates[key];
    }
    this.isSelectingCell = false; // Reset the flag
    return;
  }

  // Handle selection logic: Find the closest selected date in the same row and area
  let closestDay = null;
  for (let d = 1; d <= 30; d++) {
    const otherKey = `${rowId}-${d}`;
    if (this.selectedDates[otherKey] === this.selectedHotelLocation) {
      if (!closestDay || Math.abs(d - day) < Math.abs(closestDay - day)) {
        closestDay = d;
      }
    }
  }

  // If there's a closest day found, select all days in between
  if (closestDay !== null) {
    const start = Math.min(day, closestDay);
    const end = Math.max(day, closestDay);
    for (let d = start; d <= end; d++) {
      this.selectedDates[`${rowId}-${d}`] = this.selectedHotelLocation;
    }
  } else {
    this.selectedDates[key] = this.selectedHotelLocation;
  }

  // Reset the flag once the selection process is completed
  this.isSelectingCell = false;
},





getCellText(rowId, day) {
  const location = this.selectedDates[`${rowId}-${day}`];
  if (location) {
    const roomSetting = this.roomSettings[rowId];

    if (location === 'shiftingMakkah') {
      // Calculate number of rooms for Shifting Makkah based on Main Makkah pilgrims
      const mainMakkahPilgrims = this.calculatePilgrims('mainMakkah', rowId);
      const shiftingRoomType = roomSetting?.shiftingRoomType || 'quadruple';
      const roomCapacity = this.getRoomCapacity(shiftingRoomType);
      const numberOfRooms = mainMakkahPilgrims > 0 ? Math.ceil(mainMakkahPilgrims / roomCapacity) : 0;
      return numberOfRooms > 0 ? numberOfRooms.toString() : '-';
    } else {
      // Main Makkah or Madinah
      const numberOfRooms = roomSetting?.numberOfRooms || 0;
      return numberOfRooms > 0 ? numberOfRooms.toString() : '-';
    }
  } else {
    return '-';
  }
},

getCellClass(rowId, day) {
      const location = this.selectedDates[`${rowId}-${day}`];
      return location ? location : '';
    },

    calculate() {
      this.summaryData = [];
      let map = new Map();

      for (const [key, area] of Object.entries(this.selectedDates)) {
        const [rowId, day] = key.split('-');
        // Check if the row is confirmed
    if (!this.confirmedRows[rowId]) {
      continue; // Skip unconfirmed rows
    }
        const date = new Date(2024, 5, parseInt(day));
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
      alert("Adjusting Shifting Makkah room type to match Main Makkah due to non-integer room count.");
      shiftingRoomType = this.roomSettings[rowId].mainRoomType;
      // Update the shiftingRoomType in roomSettings
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
  return date.toLocaleDateString('en-GB', {
    day: '2-digit',
    month: '2-digit'
  });
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
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
th, td {
  padding: 5px;
  text-align: center;
  cursor: pointer;
}
th {
  background-color: #f2f2f2;
  cursor: default;
}
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
  width: 30px;
}



</style>

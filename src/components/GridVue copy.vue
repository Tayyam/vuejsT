
<template>
  <div>
    <select v-model="selectedHotelLocation">
      <option value="">Select Location</option>
      <option value="mainMakkah">Main Makkah</option>
      <option value="shiftingMakkah">Shifting Makkah</option>
      <option value="madinah">Madinah</option>
    </select>
    <br>
    <button @click="addRoomLine">Add Room Line</button>
    <br><br>
    <table>
      <tr>
        <th>ID</th>
        <th v-for="day in 30" :key="day">{{ day }}/6</th>
      </tr>
      <tr v-for="(line, index) in roomLines" :key="index">
        <td>{{ index + 1 }}</td>
        <td v-for="day in 30" :key="day" @click="selectCell(index + 1, day)" :class="getCellClass(index + 1, day)">{{ getCellText(index + 1, day) }}</td>
      </tr>
    </table>

    <br>
    Current Row ID: <input type="number" v-model.number="currentRowId" min="1" :max="roomLines.length" value="1">
    <br>
    Room Type (Main Makkah & Madinah): 
  <select v-model="currentRoomSetting.mainRoomType">
    <option value="">Select Room Type</option>
    <option value="double">Double (2 pilgrims)</option>
    <option value="triple">Triple (3 pilgrims)</option>
    <option value="quadruple">Quadruple (4 pilgrims)</option>
  </select>

  Room Type (Shifting Makkah): 
  <select v-model="currentRoomSetting.shiftingRoomType">
    <option value="">Select Room Type</option>
    <option value="double">Double (2 pilgrims)</option>
    <option value="triple">Triple (3 pilgrims)</option>
    <option value="quadruple">Quadruple (4 pilgrims)</option>
  </select>
    Number of Rooms: 
<input type="number" v-model.number="currentRoomSetting.numberOfRooms" min="1">    <br><br>
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
      roomSettings: {},
      currentRowId: 1,
      summaryData: [],
    };
  },

  computed: {
    currentRoomSetting() {
      // Initialize if undefined
      this.ensureRoomSettingsExist(this.currentRowId);
      return this.roomSettings[this.currentRowId];
    }
  },

  watch: {
    currentRowId(newRowId) {
      this.ensureRoomSettingsExist(newRowId);
    }
  },

  methods: {
    ensureRoomSettingsExist(rowId) {
      if (!this.roomSettings[rowId]) {
        this.roomSettings[rowId] = {
          mainRoomType: '',
          shiftingRoomType: '',
          numberOfRooms: 1
        };
      }
    },

    addRoomLine() {
      const newLineIndex = this.roomLines.push(null) - 1;
      this.ensureRoomSettingsExist(newLineIndex + 1);
    },
    selectCell(rowId, day) {
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
    const numberOfRooms = roomSetting?.numberOfRooms || 0;
    return numberOfRooms > 0 ? numberOfRooms.toString() : '-';
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
    const date = new Date(2024, 5, parseInt(day));
    const mapKey = `${area}-${rowId}`;

    if (!map.has(mapKey)) {
      map.set(mapKey, []);
    }
    map.get(mapKey).push(date);
  }

  map.forEach((dates, key) => {
    const [area, rowId] = key.split('-');

    if (area !== 'shiftingMakkah') {
      this.pushSummaryData(area, rowId, dates);
      
      // Calculate shiftingMakkah details
      if (area === 'mainMakkah' || area === 'madinah') {
        const roomCapacity = this.getRoomCapacity(this.roomSettings[rowId]?.mainRoomType);
        const pilgrims = roomCapacity * (this.roomSettings[rowId]?.numberOfRooms || 0);
        const shiftingRoomCapacity = this.getRoomCapacity(this.currentRoomSetting.shiftingRoomType);
        const requiredRooms = Math.ceil(pilgrims / shiftingRoomCapacity);

        // Find the dates for the shiftingMakkah period
        const shiftingMakkahKey = `shiftingMakkah-${rowId}`;
        const shiftingDates = map.get(shiftingMakkahKey);
        if (shiftingDates) {
          shiftingDates.sort((a, b) => a - b);
          let startDate = shiftingDates[0];
          let endDate = shiftingDates[shiftingDates.length - 1];

          this.summaryData.push({
            id: this.summaryData.length + 1,
            area: 'Shifting Makkah',
            from: this.formatDate(startDate),
            to: this.formatDate(endDate),
            room: rowId,
            roomType: this.currentRoomSetting.shiftingRoomType,
            numberOfRooms: requiredRooms,
            pilgrims,
          });
        }
      }
    }
  });
},

pushSummaryData(area, rowId, dates) {
  dates.sort((a, b) => a - b);
  let startDate = dates[0];
  let endDate = dates[dates.length - 1];

  const formattedStartDate = this.formatDate(startDate);
  const formattedEndDate = this.formatDate(endDate);
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

  

  mounted() {
    for (let i = 0; i < 10; i++) {
      this.addRoomLine();
    }
  }
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
</style>

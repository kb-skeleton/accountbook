<template>
    <div>
      <div class="Tab">
        <Tab @tab-selected="filterEvents"/>
      </div>
      <div class="calendar">
        <FullCalendar :options="calendarOptions" />
      </div>
    </div>
  </template>
  
  <script>
  import Tab from "@/components/Tab.vue";
  import { ref, onMounted, reactive } from 'vue';
  import FullCalendar from '@fullcalendar/vue3';
  import dayGridPlugin from '@fullcalendar/daygrid';
  import interactionPlugin from '@fullcalendar/interaction';
  import axios from 'axios';
  
  export default {
    components: {
      FullCalendar, Tab
    },
    setup() {
      const calendarOptions = reactive({
        plugins: [dayGridPlugin, interactionPlugin],
        initialView: 'dayGridMonth',
        dateClick: handleDateClick,
        events: []
      });
  
      const allEvents = ref([]);
  
      onMounted(() => {
        requestList();
      });
  
      const requestList = async () => {
        try {
          const response = await axios.get('http://localhost:3000/data');
          const events = response.data.map(event => ({
            title: `${event.amount.toLocaleString()} ${event.type === 'income' ? '수입' : '지출'}`,
            start: formatDate(event.date),
            type: event.type // 이벤트 타입 추가
          }));
          allEvents.value = events;
          calendarOptions.events = events; // 초기값은 전체 데이터를 표시
        } catch (error) {
          console.error('Error fetching events:', error);
        }
      };
  
      const filterEvents = (type) => {
        if (type === 'all') {
          calendarOptions.events = allEvents.value;
        } else {
          calendarOptions.events = allEvents.value.filter(event => event.type === type);
        }
      };
  
      const formatDate = (date) => {
        const d = new Date(date);
        const year = d.getFullYear();
        const month = String(d.getMonth() + 1).padStart(2, '0');
        const day = String(d.getDate()).padStart(2, '0');
        return `${year}-${month}-${day}`;
      };
  
      function handleDateClick(arg) {
        alert('date click! ' + arg.dateStr);
      }
  
      return {
        calendarOptions,
        filterEvents
      };
    }
  };
  </script>
  
  <style scoped>
  .calendar {
    background-color: #F1F8E8;
  }
  </style>
  
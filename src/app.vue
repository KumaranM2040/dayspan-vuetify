<template>
  <v-app id="dayspan" v-cloak>

    <ds-calendar-app ref="app"
      :calendar="calendar"
      :read-only="readOnly"
      @change="saveState">

      <template slot="title">
        DaySpan
      </template>

      <template slot="menuRight">
        <v-btn icon large href="https://github.com/ClickerMonkey/dayspan-vuetify" target="_blank">
          <v-avatar size="32px" tile>
            <img src="https://simpleicons.org/icons/github.svg" alt="Github">
          </v-avatar>
        </v-btn>
      </template>

      <template slot="eventPopover" slot-scope="slotData">
         <ds-calendar-event-popover
          v-bind="slotData"
          :read-only="readOnly"
          @finish="saveState"
        ></ds-calendar-event-popover>
      </template>

      <template slot="eventCreatePopover" slot-scope="{placeholder, calendar, close}">
        <ds-calendar-event-create-popover
          :calendar-event="placeholder"
          :calendar="calendar"
          :close="$refs.app.$refs.calendar.clearPlaceholder"
          @create-edit="$refs.app.editPlaceholder"
          @create-popover-closed="saveState"
        ></ds-calendar-event-create-popover>
      </template>

      <template slot="eventTimeTitle" slot-scope="{calendarEvent, details}">
        <div>
          <v-icon class="ds-ev-icon"
            v-if="details.icon"
            size="14"
            :style="{color: details.forecolor}">
            {{ details.icon }}
          </v-icon>
          <strong class="ds-ev-title">{{ details.title }}</strong>
        </div>
        <div class="ds-ev-description">{{ getCalendarTime( calendarEvent ) }}</div>
      </template>

      <template slot="drawerBottom">
        <v-container fluid>
          <v-layout wrap align-center>
            <v-flex xs12>
              <v-checkbox box
                label="Read Only?"
                v-model="readOnly"
              ></v-checkbox>
            </v-flex>
            <v-flex xs12>
              <v-select
                label="Language"
                :items="locales"
                v-model="currentLocale"
                @input="setLocale"
              ></v-select>
            </v-flex>
          </v-layout>
        </v-container>
      </template>

    </ds-calendar-app>

  </v-app>
</template>

<script>
import { dsMerge } from './functions';
import { Calendar, Weekday, Month, Sorts } from 'dayspan';
import * as moment from 'moment';

export default {

  name: 'dayspan',

  data: vm => ({
    storeKey: 'dayspanState',
    calendar: Calendar.months(),
    readOnly: false,
    currentLocale: vm.$dayspan.currentLocale,
    locales: [
      { value: 'en', text: 'English' },
      { value: 'fr', text: 'French' },
      { value: 'de', text: 'German' },
      { value: 'nl', text: 'Dutch' },
      { value: 'ca', text: 'Catalan' }
    ],
    defaultEvents: [

    ]
  }),

  mounted()
  {
    window.app = this.$refs.app;

    this.loadState();
  },

  methods:
  {
    getCalendarTime(calendarEvent)
    {
      let sa = calendarEvent.start.format('a');
      let ea = calendarEvent.end.format('a');
      let sh = calendarEvent.start.format('h');
      let eh = calendarEvent.end.format('h');

      if (calendarEvent.start.minute !== 0)
      {
        sh += calendarEvent.start.format(':mm');
      }

      if (calendarEvent.end.minute !== 0)
      {
        eh += calendarEvent.end.format(':mm');
      }

      return (sa === ea) ? (sh + ' - ' + eh + ea) : (sh + sa + ' - ' + eh + ea);
    },

    setLocale(code)
    {
      moment.lang(code);

      this.$dayspan.setLocale(code);
      this.$dayspan.refreshTimes();

      this.$refs.app.$forceUpdate();
    },

    async saveState()
    {
      let state = this.calendar.toInput(true);
      let json = JSON.stringify(state);
      debugger

       try {
        await this.$http.post('/relays',json);
      }
      catch(ex){
        // eslint-disable-next-line
        console.log(ex);
      }

      localStorage.setItem(this.storeKey, json);
    },

    async loadState()
    {
      let state = {};
      debugger
      try
      {

          let result = null;
            try {
            result = await this.$http.get('/relays');
          }
          catch(ex){
            // eslint-disable-next-line
            console.log(ex);
          }
        let savedState = JSON.parse(result);
        //let savedState = JSON.parse(localStorage.getItem(this.storeKey));

        if (savedState)
        {
          state = savedState;
          state.preferToday = false;
        }
      }
      catch (e)
      {
        console.log( e );
      }

      if (!state.events || !state.events.length)
      {
        state.events = this.defaultEvents;
      }

      let defaults = this.$dayspan.getDefaultEventDetails();

      state.events.forEach(ev =>
      {
        ev.data = dsMerge( ev.data, defaults );
      });

      this.$refs.app.setState( state );
    }
  }
}
</script>

<style>

body, html, #app {
  width: 100%;
  height: 100%;
}

</style>

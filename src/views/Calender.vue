<template>
<v-container>
<div class="text-center">
    <vc-date-picker v-model="date" mode="date" :attributes='attributes' @dayclick="onDayClick" @update:to-page='onUpdate' />
<v-dialog
      v-model="dialog"
      width="500"
    >
         
       
      <v-card>
        <v-card-title class="headline grey lighten-2">
          {{selectedDate}}
        </v-card-title>

        <v-card-text>
    <v-list dense>
      <v-list-item-group
        v-model="selectedItem"
        color="primary"
      >
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          @click='getEvent(i)'
        >
          <v-list-item-content>
            <v-list-item-title v-text="item.title"></v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list-item-group>
    </v-list>        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="primary"
            text
            @click="dialog = false"
          >
            とじる
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>


    <v-dialog
      v-model="dialog2"
      width="500"
    >
         
       
      <v-card>
        <v-card-title class="headline grey lighten-2">
          {{eventTitle}}
        </v-card-title>

        <v-card-text>
          {{eventBody}}
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-btn
            color="error"
            text
            @click="deleteEvent"
          >
            削除
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn
            color="primary"
            text
            @click="dialog2 = false"
          >
            とじる
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    </div>
</v-container>
</template>

<script>
const axios = require('axios');
const endpoint = 'https://pf2hhkoc2k.execute-api.ap-northeast-1.amazonaws.com/prod';
  
export default {
  name: 'Calender',
  data: () => ({
      date: new Date(),
      attributes: [],
      begin: '',
      end: '',
      schedule: new Map(),
      events: new Map(),
      selectedItem: 1,
      selectedDate: '',
      eventUuid: '',
      eventTitle: '',
      eventBody: '',
      items: [],
      dialog: false,
      dialog2: false,
    }),
    methods: {
    onDayClick(day) {
        console.log(day)
        if (this.schedule.has(day.id)) {
            this.items = this.schedule.get(day.id)
            this.selectedDate = day.id
            console.log(this.items)
            this.dialog = true
            //console.log(events.map((e) => e.title))
        }
    },
    onUpdate(page) {
        console.log(page);
        this.begin = page.year + '-' +  ('0' + page.month).slice(-2) + '-01'
        this.end = page.year + '-' +  ('0' + page.month).slice(-2) + '-' + (new Date(page.year, page.month, 0)).getDate()
        
        this.fetch()
    },
    fetch: async function() {
        const params = {
            id : this.$route.params.id,
            begin : this.begin,
            end : this.end
        };
        const attrs = [];
        const map = new Map()
        await axios.get(endpoint + '/schedule/', {params: params}).then(res =>  {
            console.log(res.data);
            res.data.forEach(date => {
                  attrs.push({
                        dot: 'red',
                        dates: new Date(date.date),
                      })
                  map.set(date.date, date.events);
            });
            this.attributes = attrs;
            this.schedule = map
        });

    },
    getEvent(index) {
      const uuid = this.items[index].uuid
        console.log(uuid)
        if (this.events.has(uuid)) {
          const event = this.events.get(uuid)
          this.eventUuid = uuid
          this.eventTitle = event.title;
          this.eventBody = event.body;
          this.dialog2 = true;
          
        } else {
          const params = {uuid : uuid};
          axios.get(endpoint + '/event/', {params: params}).then(res =>  {
              console.log(res.data.body)
              this.eventUuid = uuid
              this.eventTitle = res.data.title;
              this.eventBody = res.data.body;
              this.dialog2 = true;
              this.events.set(uuid, res.data);
          });
        }
    },
    deleteEvent: async function() {
      const params = {
        id: this.$route.params.id,
        date: this.selectedDate,
        uuid: this.eventUuid
      }
      console.log("selected date : " + this.selectedDate)
      await axios.delete(endpoint + '/event/', {params: params}).then(res =>  {
              console.log(res.data)
            })
      this.dialog2 = false;
      await this.fetch()

      if (this.schedule.has(params.date)) {
        this.items = this.schedule.get(params.date)
        console.log(params.date)
        console.log(this.items)
      } else {
        this.dialog = false
      }
              
   
    }
  },
};
</script>
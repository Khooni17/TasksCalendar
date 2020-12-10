<template>
    <div id="app">
        <div class="header">
            <div class="header-cont">
                <div class="header-month header-item">
                    <p class="day-number month">{{ month < 10 ? `0${month}` : month }}</p>
                </div>
                <div class="header-item" v-for="(day, index) in days" :key="index">
                    <p class="day-title">{{ day.shortTitle }}</p>
                    <p class="day-number">{{ day.number }}</p>
                </div>
            </div>
        </div>
        <div class="table-wrap">

            <div class="table">
                <div class="table-row" v-for="h in 24" :key="h">
                    <div class="table-cell">
                        <p>{{ formatHour(h) }}</p>
                    </div>
                    <div @drop="onDrop($event, h, day.number)"
                         @dragenter="markCell($event)"
                         @dragleave="unmarkCell($event)"
                         class="table-cell droppable"
                         @dragover.prevent
                         @dragenter.prevent
                         @dblclick.prevent="addNewEvent(h, day.number)"
                         v-for="(day, index) in days" :key="index">
                        <div @dragstart="onDragStart($event, ev)"
                             class="table-event draggable"
                             draggable="true"
                             v-for="ev in calcEvent(day.number, h)"
                             @dblclick.stop="startEditing(ev.id)"
                             :key="ev.id">
                            <input type="text"
                                   class="event-edit"
                                   :id="ev.id"
                                   v-if="eventEdit === ev.id"
                                   v-model="ev.title"
                                   @keyup.enter.stop="stopInput"
                                   @keyup.esc.stop="stopInput"
                                   @focusout.stop="stopInput"
                            >
                            <p class="event-title" v-else>{{ ev.title }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>


export default {
    name: 'app',
    data() {
        return {
            startDate: new Date('03-30-2020'),
            month: '',
            eventEdit: '',
            events: [
                {"title": "Заголовок 1", "startDate": 1585699200, "endDate": 1585702800},
                {"title": "Заголовок 2", "startDate": 1585706400, "endDate": 1585710000},
                {"title": "Заголовок 3", "startDate": 1585789200, "endDate": 1585792800},
                {"title": "Заголовок 4", "startDate": 1586307600, "endDate": 1586311200},
                {"title": "Заголовок 5", "startDate": 1585791500, "endDate": 1585795100},
                {"title": "Заголовок 6", "startDate": 1585706400, "endDate": 1585710000}
            ],
            days: []
        }
    },
    methods: {
        stopInput(){
            this.eventEdit = '';
            this.events = this.events.filter((ev) => ev.title !== '');
        },
        startEditing(id){
            this.eventEdit = id;
            setTimeout(() => {
                document.getElementById(this.eventEdit).focus();
            }, 1);
        },
        addNewEvent(h, d){
            let newD = new Date(this.startDate.getTime());
            newD.setDate(d)
            newD.setHours(h);

            this.events.push({
                title: "",
                numHour: h,
                numDay: d,
                fullDate: newD,
                startDate: newD.getTime()/1000,
                endDate: new Date(newD.getTime() + 1000 * 60 * 60).getTime() / 1000,
                id: this.events.reduce((prev, curr) => prev.id > curr.id ? prev : curr).id+1
            })
            this.eventEdit = this.events[this.events.length - 1].id;
            setTimeout(() => {
                document.getElementById(this.eventEdit).focus();
            }, 1);

        },
        markCell(e){
            e.target.classList.add('marked-cell');
        },
        unmarkCell(e){
            e.target.classList.remove('marked-cell');
        },

        calcEvent(numDay, numHour) {
            return this.events.filter(el => el['numDay'] === numDay && el['numHour'] === numHour);
        },
        formatHour(h) {
            if (h < 10) {
                return `0${h}:00`
            }
            return `${h}:00`
        },


        onDragStart(e, element) {
            e.dataTransfer.dropEffect = 'move';
            e.dataTransfer.effectAllowedv = 'move';
            e.dataTransfer.setData('elHours', element.numHour);
            e.dataTransfer.setData('elDay', element.numDay);
            e.dataTransfer.setData('id', element.id);
        },

        onDrop(e, hour, day) {
            this.unmarkCell(e);
            //console.log(e.dataTransfer.getData('el'));
            const elHours = parseInt(e.dataTransfer.getData('elHours'));
            const elDay = parseInt(e.dataTransfer.getData('elDay'));
            const id = parseInt(e.dataTransfer.getData('id'));

            this.events = this.events.map(el => {
                if (el.id === id) {
                    el['numHour'] = hour;
                    el['numDay'] = day;
                    el['fullDate'].setDate(day);
                    el['fullDate'].setHours(hour);
                    el['startDate'] = el['fullDate'].getTime() / 1000;
                    el['endDate'] = new Date(el['fullDate'].getTime() + 1000 * 60 * 60).getTime() / 1000;
                }
                return el;
            })
            /*            const el = e.dataTransfer.getData('el');
            el['numHour'] = hour;
            el['numDay'] = day;*/
        },

    },
    computed: {},
    created() {
        this.events = this.events.map((ev, index) => {
            ev.id = index;
            ev.fullDate = new Date(ev.startDate * 1000);
            ev.numDay = ev.fullDate.getDate();
            ev.numHour = ev.fullDate.getHours();

            return ev;
        })
        for (let i = 0; i < 7; i++) {
            this.days.push({
                shortTitle: ['ПН', 'ВТ', 'СР', 'ЧТ', 'ПТ', 'СБ', 'ВС'][new Date(this.startDate.getTime() + 1000 * 60 * 60 * 24 * i).getDay()],
                number: new Date(this.startDate.getTime() + 1000 * 60 * 60 * 24 * i).getDate()
            });
        }
        this.month = ['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC'][this.startDate.getMonth()];
    },
    watch: {
        startDate() {
            for (let i = 0; i < 7; i++) {
                this.days.push({
                    shortTitle: ['ПН', 'ВТ', 'СР', 'ЧТ', 'ПТ', 'СБ', 'ВС'][new Date(this.startDate.getTime() + 1000 * 60 * 60 * 24 * i).getDay()],
                    number: new Date(this.startDate.getTime() + 1000 * 60 * 60 * 24 * i).getDate()
                });
            }
            this.month = this.startDate.getMonth();
        }
    }

}
</script>

<style lang="scss">
@import "styles_fonts";

$font-Inter: Inter, 'Inter';

$grey: #cacaca;

body {
    margin: 0;
}

#app {
    width: 100vw;
    height: 100vh;


    .header {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: flex-end;
        height: 100px;

        p {
            font-family: $font-Inter;
            color: grey;
            margin: 0;
        }

        .header-cont {
            height: 100%;
            width: calc(100%);
            display: grid;
            grid-template-columns: 75px repeat(7, 1fr);


            .header-item {
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;

                .day-title {
                    font-size: 18px;
                }


                .day-number {
                    margin-top: 7px;
                    font-size: 27px;
                }

                .month {
                    font-size: 21px;
                    font-weight: 600;
                }

            }

            .header-month {
                .day-number {
                    margin-top: 0;
                }
            }
        }
    }

    .table-wrap {
        width: 100%;
        height: calc(100% - 100px);

        p {
            font-family: $font-Inter;
        }

        .table {
            width: 100%;
            height: calc(100% - 50px);
            max-height: calc(100% - 50px);
            overflow-y: auto;

            &::-webkit-scrollbar {
                width: 7px;
                border: none;
            }

            &::-webkit-scrollbar-thumb {
                background: $grey;
                border-radius: 5px;
            }
            //background: green;

            .table-row {
                user-select: none;
                border-right: none;
                min-height: 50px;
                display: grid;
                grid-template-columns: 75px repeat(7, 1fr);

                .table-cell {
                    border-left: 1px solid $grey;
                    border-top: 1px solid $grey;
                    border-bottom: 1px solid $grey;
                    overflow: hidden;

                    &:nth-child(1){
                        text-align: center;
                    }

                    .table-event {
                        font-size: 14px;
                        background: #ecefc6;
                        width: 90%;
                        max-width: 90%;
                        cursor: pointer;
                        margin-bottom: 4px;

                        .event-edit {
                            background: #ecefc6;
                            width: 100%;
                            outline: none;
                            border: 1px solid $grey;
                            box-shadow: 0 0 15px 1px $grey;
                            height: 14px;
                            padding: 4px 4px;
                            font-family: $font-Inter;
                        }

                        .event-title, {
                            font-family: $font-Inter;
                            max-width: 100%;
                            text-overflow: ellipsis;
                            display: -webkit-box;
                            -webkit-line-clamp: 2;
                            overflow: hidden;
                            white-space: pre-wrap;
                            -webkit-box-orient: vertical;
                            margin: 0;
                            padding: 2px 5px;
                            color: #1f1f1f;
                        }
                    }

                    &:nth-child(1) {
                        position: relative;
                        border: none;

                        &:before {
                            content: " ";
                            position: absolute;
                            top: 0;
                            display: block;

                            bottom: 0;
                            right: 0;
                            height: calc(100% - 1px);
                            border-bottom: 2px solid $grey;
                            width: 50%;
                        }
                    }
                }


                .marked-cell {
                    border-color: #9ecaed;
                    box-shadow: 0 0 15px 1px #9ecaed;
                }
            }
        }

    }

}

</style>

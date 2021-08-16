<template>
  <div class="full-height">
    <v-navigation-drawer id="navigation" absolute left app>
      <v-list>
        <v-list-item-group
          v-model="selectedView"
          @change="viewSelectionChanged"
          mandatory
        >
          <v-list-item>
            <v-list-item-title> My Schedule </v-list-item-title></v-list-item
          >
          <v-divider></v-divider>
          <v-list-item-title>
            <v-subheader
              >Groups
              <v-spacer></v-spacer>
              <v-btn x-small @click="joinGroupDialog">join</v-btn>
              <v-btn x-small @click="createGroupDialog">create</v-btn>
            </v-subheader>
          </v-list-item-title>
          <v-list-item v-for="(group, index) in groups" :key="group.id">
            <template v-slot:default="{ active }">
              <v-list>
                <v-list-item-title>
                  {{ group.title }}
                  <v-list-item-action v-if="active">
                    <v-btn x-small icon @click="leaveGroup(index)">
                      <v-icon>mdi-exit-to-app</v-icon>
                    </v-btn>
                  </v-list-item-action>
                </v-list-item-title>
                <v-list-item-subtitle v-if="active">
                  Members
                </v-list-item-subtitle>
                <template v-if="active">
                  <v-list-item
                    v-for="user in group.users"
                    :key="user.id"
                    disabled
                  >
                    <v-list-item-avatar>
                      <v-avatar :color="getColorForUser(user.id)" size="24">
                      </v-avatar>
                    </v-list-item-avatar>
                    {{ user.username }}
                  </v-list-item>
                </template>
              </v-list>
            </template>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <v-main class="full-height" app>
      <v-toolbar dense>
        <v-toolbar-title>
          {{ getViewTitle }}
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn
          class="mx-1"
          @click="showCreateEventDialog"
          :disabled="!isUserView"
          >create</v-btn
        >
        <v-btn
          class="mx-1"
          @click="showEditEventDialog"
          :disabled="!hasSelectedEvent || !isUserView"
        >
          edit
        </v-btn>
        <v-btn
          class="mx-1"
          @click="deleteSelectedEvent"
          :disabled="!hasSelectedEvent || !isUserView"
        >
          delete
        </v-btn>
        <v-btn class="mx-1" @click="refresh">refresh</v-btn>

        <v-spacer></v-spacer>
      </v-toolbar>
      <v-sheet id="calendar-container">
        <FullCalendar
          ref="calendar"
          id="calendar"
          :options="calendarOptions"
        ></FullCalendar>
      </v-sheet>

      <!-- Pop-up Dialogs -->
      <EventDialog
        ref="EventDialog"
        :visible="showEventDialog"
        :mode="eventDialogMode"
        :initialEvent="this.selectedEvent"
        @close="hideEventDialog"
        @create-event="createOrUpdateEvent"
        @edit-event="createOrUpdateEvent"
      ></EventDialog>
      <GroupDialog
        ref="GroupDialog"
        :visible="showGroupDialog"
        :mode="groupDialogMode"
        @close="hideGroupDialog"
        @create-group="createOrUpdateGroup"
        @join-group="joinGroup"
        @edit-group="createOrUpdateGroup"
      ></GroupDialog>
      <v-snackbar v-model="snackbar.visible" timout="4000">{{
        snackbar.msg
      }}</v-snackbar>
    </v-main>
  </div>
</template>

<script>
import FullCalendar from "@fullcalendar/vue";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";
import EventDialog from "@/Components/EventDialog.vue";
import GroupDialog from "../Components/GroupDialog.vue";
import { endpoints } from "../endpoints";

export default {
  name: "Scheduler",
  components: {
    FullCalendar,
    EventDialog,
    GroupDialog,
  },
  data() {
    return {
      selectedView: 0,
      groups: [],
      calendarOptions: {
        plugins: [
          dayGridPlugin,
          timeGridPlugin,
          interactionPlugin, // needed for dateClick
        ],
        headerToolbar: {
          left: "prev,next today",
          center: "title",
          right: "dayGridMonth,timeGridWeek,timeGridDay",
        },
        initialView: "timeGridWeek",
        events: this.events,
        editable: true,
        selectable: false,
        dayMaxEvents: true,
        weekends: true,
        height: "100%",
        eventClick: this.eventClick,
        eventChange: this.eventChange,
      },
      colors: [
        "#008B8B", // Dark Cyan
        "#2F4F4F", // DarkSlateGrey
        "#228B22", // Forest Green
        "#20B2AA", // Light Sea Green
        "#191970", // Midnight Blue
        "#1E90FF", // DodgerBlue
        "#8A2BE2", // Blue Violet
        "#DC143C", // Crimson
        "#FF1493", // Deep pink
        "#FF6347", // Tomato
        "#B22222", // Fire Brick
        "#DAA520", // Golden Rod
      ],
      selectedEvent: undefined,
      showEventDialog: false,
      eventDialogMode: 0,
      showGroupDialog: false,
      groupDialogMode: 0,
      snackbar: { visible: false, msg: "" },
    };
  },
  mounted() {
    this.fetchGroups();
    this.fetchEvents();
  },
  computed: {
    calendarEditable() {
      return this.selectedView == 0;
    },
    getViewTitle() {
      if (this.selectedView != 0) {
        return this.groups[this.selectedView - 1].title;
      }
      return "My Schedule";
    },
    isUserView() {
      return this.selectedView == 0;
    },
    hasSelectedEvent() {
      return !!this.selectedEvent;
    },
  },
  methods: {
    refresh() {
      this.fetchGroups();
      this.fetchEvents();
    },
    eventClick(eventClickInfo) {
      if (this.selectedView == 0) {
        if (this.selectedEvent) {
          this.selectedEvent.setProp("backgroundColor", undefined);
        }
        this.selectedEvent = eventClickInfo.event;
        this.selectedEvent.setProp("backgroundColor", "#15b500");
      }
    },
    eventChange(changeInfo) {
      console.log("event change");
      const newEvent = changeInfo.event.toPlainObject({
        collapseExtendedProps: true,
      });
      const oldEvent = changeInfo.oldEvent.toPlainObject({
        collapseExtendedProps: true,
      });

      const noneStylePropChanged =
        newEvent.title != oldEvent.title ||
        newEvent.start != oldEvent.start ||
        newEvent.end != oldEvent.end ||
        newEvent.daysOfWeek != oldEvent.daysOfWeek ||
        newEvent.recurrenceStart != oldEvent.recurrenceStart ||
        newEvent.recurrenceEnd != oldEvent.recurrenceEnd;

      if (noneStylePropChanged) {
        this.createOrUpdateEvent(newEvent, false);
      }
    },
    // Server Requests
    fetchGroups() {
      console.log("fetching groups");
      const token = localStorage.getItem("schedulerJWT");
      this.axios
        .get(endpoints.groups, {
          headers: {
            "content-type": "application/json",
            authorization: token,
          },
        })
        .then((res) => {
          console.log("groups");
          console.log(res);
          this.groups = res.data;
        })
        .catch((err) => {
          console.log(err.response);
          if (err.response.data.status == "403") {
            this.$router.push({ path: "/" });
          } else {
            this.showSnackBar(
              `Error: ${err.response.status}: ${err.response.data}`
            );
          }
        });
    },
    fetchEvents() {
      console.log("get events");
      const token = localStorage.getItem("schedulerJWT");

      let headers = {
        "content-type": "application/json",
        authorization: token,
      };
      const isGroupEvents = this.selectedView != 0;
      if (isGroupEvents) {
        headers.group_title = this.groups[this.selectedView - 1].title;
      }
      this.axios
        .get(endpoints.events, { headers })
        .then((res) => {
          console.log(res);
          this.clearEvents();
          this.addEvents(res.data, isGroupEvents);
        })
        .catch((err) => {
          console.error(err);
          if (err.response.data.status == "403") {
            this.$router.push({ path: "/" });
          } else {
            this.showSnackBar(
              `Error: ${err.response.status}: ${err.response.data}`
            );
          }
        });
    },

    // Event Related Functions
    hideEventDialog() {
      this.showEventDialog = false;
    },
    showCreateEventDialog() {
      this.eventDialogMode = 0;
      this.showEventDialog = true;
    },
    showEditEventDialog() {
      this.eventDialogMode = 1;
      this.showEventDialog = true;
      this.$refs.EventDialog.setEvent(
        this.selectedEvent.toPlainObject({ collapseExtendedProps: true })
      );
    },
    eventDialogRules() {
      return !!this.eventDialogData.name;
    },
    createOrUpdateEvent(event, addToLocal = true) {
      const token = localStorage.getItem("schedulerJWT");
      this.axios
        .post(endpoints.event, event, {
          headers: {
            "content-type": "application/json",
            authorization: token,
          },
        })
        .then((res) => {
          if (res.status == 200) {
            const newEvent = res.data;
            if (addToLocal) {
              this.removeEvent(newEvent.id);
              this.addEvent(newEvent);
            }
          } else {
            this.showSnackBar(res.statusText);
          }
        })
        .catch((err) => {
          console.log(err);
          if (err.response.data.status == "403") {
            this.$router.push({ path: "/" });
          } else {
            this.showSnackBar(
              `Error: ${err.response.status}: ${err.response.data}`
            );
          }
        });
    },
    deleteSelectedEvent() {
      if (!this.selectedEvent) {
        return;
      }
      const token = localStorage.getItem("schedulerJWT");
      this.axios
        .post(
          endpoints.removeEvent,
          { eventid: this.selectedEvent.id },
          {
            headers: {
              "content-type": "application/json",
              authorization: token,
            },
          }
        )
        .then((res) => {
          console.log("deleted");
          console.log(res);
          this.removeEvent(this.selectedEvent.id);
          this.selectedEvent = undefined;
        })
        .catch((err) => {
          console.error(err);
          if (err.response.data.status == "403") {
            this.$router.push({ path: "/" });
          } else {
            this.showSnackBar(
              `Error: ${err.response.status}: ${err.response.data}`
            );
          }
        });
    },
    addEvent(event) {
      this.$refs.calendar.getApi().addEvent(event);
    },
    removeEvent(eventid) {
      const event = this.$refs.calendar.getApi().getEventById(eventid);
      if (event) {
        event.remove();
      }
    },
    addEvents(events, isGroupEvents = false) {
      events.forEach((ev) => {
        if (isGroupEvents) {
          ev.backgroundColor = this.getColorForUser(ev.userid);
        }
        this.$refs.calendar.getApi().addEvent(ev);
      });
    },
    clearEvents() {
      let events = this.$refs.calendar.getApi().getEvents();
      events.forEach((ev) => {
        ev.remove();
      });
    },

    // Group Related Functions
    createOrUpdateGroup(group) {
      const token = localStorage.getItem("schedulerJWT");
      this.axios
        .post(endpoints.group, group, {
          headers: {
            "content-type": "application/json",
            authorization: token,
          },
        })
        .then((res) => {
          console.log(res);
          this.groups.push(res.data);
          this.$refs.GroupDialog.setVisible(false);
          this.$refs.GroupDialog.reset();
        })
        .catch((err) => {
          console.log(err.response);
          if (err.response.data.status == "403") {
            this.$router.push({ path: "/" });
          } else {
            this.showSnackBar(
              `Error: ${err.response.status}: ${err.response.data}`
            );
          }
        });
    },
    joinGroup(group) {
      const token = localStorage.getItem("schedulerJWT");
      console.log("attempting to join group");
      console.log(group);
      this.axios
        .post(endpoints.joinGroup, group, {
          headers: {
            "content-type": "application/json",
            authorization: token,
          },
        })
        .then((res) => {
          console.log(res);
          this.groups.push(res.data);
          this.$refs.GroupDialog.setVisible(false);
          this.$refs.GroupDialog.reset();
        })
        .catch((err) => {
          console.log(err.response);
          this.showSnackBar(
            `Error: ${err.response.status}: ${err.response.data}`
          );
        });
    },
    leaveGroup(groupIndex) {
      const token = localStorage.getItem("schedulerJWT");
      console.log("attempting to leave group");
      const title = this.groups[groupIndex].title;
      console.log(title);
      this.axios
        .post(
          endpoints.leaveGroup,
          { title },
          {
            headers: {
              "content-type": "application/json",
              authorization: token,
            },
          }
        )
        .then((res) => {
          console.log(res);
          console.log("left group");
          this.selectedView = 0;
          this.groups.splice(groupIndex, 1);
        })
        .catch((err) => {
          console.log(err.response);
          if (err.response.data.status == "403") {
            this.$router.push({ path: "/" });
          } else {
            this.showSnackBar(
              `Error: ${err.response.status}: ${err.response.data}`
            );
          }
        });
    },

    hideGroupDialog() {
      this.showGroupDialog = false;
    },
    joinGroupDialog() {
      this.groupDialogMode = 0;
      this.$nextTick(() => {
        this.showGroupDialog = true;
      });
    },

    createGroupDialog() {
      this.groupDialogMode = 1;
      this.$nextTick(() => {
        this.showGroupDialog = true;
      });
    },

    // Other
    viewSelectionChanged() {
      console.log(this.selectedView == 0);
      this.$refs.calendar.options.editable = this.selectedView == 0;
      this.$refs.calendar.options.selectable = this.selectedView == 0;
      this.scrambleColors();
      this.fetchEvents();
      this.selectedEvent = null;
    },
    showSnackBar(msg) {
      this.snackbar.msg = msg;
      this.snackbar.visible = true;
    },
    getColorForUser(userid) {
      const group = this.groups[this.selectedView - 1];
      if (group && group.users) {
        const userIndex = group.users.findIndex((user) => {
          return user.id == userid;
        });
        if (userIndex >= 0) {
          const color = this.colors[userIndex];
          return color;
        }
      }

      return this.colors[Math.round(Math.random() * this.colors.length)];
    },
    scrambleColors() {
      for (var i = this.colors.length - 1; i > 0; i--) {
        var j = Math.floor(Math.random() * (i + 1));
        var temp = this.colors[i];
        this.colors[i] = this.colors[j];
        this.colors[j] = temp;
      }
    },
  },
};
</script>

<style>
#scheduler-container {
  top: 0;
  bottom: 0;
}
.full-height {
  height: 100%;
}

#calendar-container {
  height: calc(100% - 64px);
}
</style>

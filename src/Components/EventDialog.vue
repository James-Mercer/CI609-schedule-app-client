<template>
  <v-dialog id="event-dialog" v-model="dialogVisible" max-width="800px">
    <v-card>
      <v-card-title>
        {{ mode == 0 ? "Create Event" : "Edit Event" }}
      </v-card-title>
      <v-card-text>
        <v-form ref="form" v-model="valid" lazy-validation>
          <v-text-field
            label="Title"
            v-model="event.title"
            :rules="[titleRule]"
            counter
            max-length="60"
          ></v-text-field>
          <v-row>
            <v-col>
              <v-text-field
                label="Start"
                v-model="startStr"
                type="datetime-local"
                :rules="[startRule]"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                label="end"
                v-model="endStr"
                type="datetime-local"
                :rules="[endRule]"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-checkbox label="Reccuring" v-model="reccuring"></v-checkbox>
            </v-col>
            <v-col class="flex-grow-1">
              <v-select
                label="Days"
                v-model="event.daysOfWeek"
                multiple
                :items="days"
                :rules="reccuring ? [daysRule] : []"
                :disabled="!reccuring"
              ></v-select>
            </v-col>
          </v-row>
          <v-row v-if="reccuring">
            <v-col>
              <v-checkbox
                label="Use Range"
                v-model="recurrenceRange"
              ></v-checkbox>
            </v-col>
            <v-col>
              <v-text-field
                label="Recurrence Range start"
                v-model="recStartStr"
                type="datetime-local"
                :disabled="!recurrenceRange"
                :rules="[startRule]"
              >
              </v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                label="Recurrence Range End"
                v-model="recEndStr"
                type="datetime-local"
                :disabled="!recurrenceRange"
                :rules="[endRule]"
              >
              </v-text-field>
            </v-col>
          </v-row>
        </v-form>
      </v-card-text>
      <v-card-actions
        ><v-spacer></v-spacer>
        <v-btn @click="emitClose" color="red">Cancel</v-btn>
        <v-btn v-if="mode == 0" @click="emitCreateEvent" color="primary">
          Create
        </v-btn>
        <v-btn v-else @click="emitEditEvent" color="primary"> Edit </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "EventDialog",
  props: {
    visible: Boolean,
    mode: Number,
    initialEvent: {
      type: Object,
      default: () => {
        return {
          title: "",
          start: new Date(),
          end: new Date(),
          startTime: new Date(),
          endTime: new Date(),
          daysOfWeek: [],
          startRecur: new Date(),
          endRecur: new Date(),
        };
      },
    },
  },
  data() {
    return {
      dialogVisible: this.initalVisible,
      event: this.initialEvent,
      valid: false,
      reccuring: false,
      recurrenceRange: false,
      days: [
        { text: "Monday", value: 1 },
        { text: "Tuesday", value: 2 },
        { text: "Wednesday", value: 3 },
        { text: "Thursday", value: 4 },
        { text: "Friday", value: 5 },
        { text: "Saturday", value: 6 },
        { text: "Sunday", value: 0 },
      ],
    };
  },
  computed: {
    startStr: {
      get: function () {
        return this.formatDate(this.event.start);
      },
      set: function (val) {
        this.event.start = new Date(val + ":00");
      },
    },
    endStr: {
      get: function () {
        return this.formatDate(this.event.end);
      },
      set: function (val) {
        this.event.end = new Date(val + ":00");
      },
    },
    recStartStr: {
      get: function () {
        return this.formatDate(this.event.startRecur);
      },
      set: function (val) {
        this.event.startRecur = new Date(val + ":00");
      },
    },
    recEndStr: {
      get: function () {
        return this.formatDate(this.event.endRecur);
      },
      set: function (val) {
        this.event.endRecur = new Date(val + ":00");
      },
    },
  },
  methods: {
    processEvent(event) {
      let eventProcessedEvent = {
        title: event.title,
      };
      if (this.recurring) {
        eventProcessedEvent.startTime = new Date(event.start) || new Date();
        eventProcessedEvent.endTime = new Date(event.end) || new Date();
      } else {
        eventProcessedEvent.start = new Date(event.start) || new Date();
        eventProcessedEvent.end = new Date(event.end) || new Date();
      }

      if (this.recurrenceRange) {
        eventProcessedEvent.startRecur = event.startRecur;
        eventProcessedEvent.endRecur = event.endRecur;
      }
      return eventProcessedEvent;
    },
    emitCreateEvent() {
      if (this.$refs.form.validate() === true) {
        this.$emit("create-event", this.processEvent(this.event));
        this.reset();
      }
    },
    emitEditEvent() {
      if (this.$refs.form.validate()) {
        this.$emit("edit-event", this.processEvent(this.event));
        this.reset();
      }
    },
    emitClose() {
      this.$emit("close");
      this.reset();
    },
    formatDate(date) {
      const addDigit = (num) => {
        return num < 10 ? `0${num}` : num;
      };
      return `${date.getFullYear()}-${addDigit(date.getMonth())}-${addDigit(
        date.getDate()
      )}T${addDigit(date.getHours())}:${addDigit(date.getMinutes())}`;
    },
    titleRule(v) {
      return !!v || "Title required";
    },
    startRule(v) {
      return (
        new Date(v + ":00").toString() !== "Invalid Date" ||
        "Start must be valid"
      );
    },
    endRule(v) {
      return (
        new Date(v + ":00").toString() !== "Invalid Date" || "End must be valid"
      );
    },
    daysRule() {
      return this.event.daysOfWeek && this.event.daysOfWeek.length > 0
        ? true
        : "Required if reccuring is set";
    },
    setEvent(event) {
      console.log(event);
      this.event = this.processEvent(event);
    },
    reset() {
      this.dialogVisible = false;
      this.recurring = false;
      this.recurrenceRange = false;
      this.event = {
        title: "",
        start: new Date(),
        end: new Date(),
        daysOfWeek: [],
        recurrenceStart: new Date(),
        recurrenceEnd: new Date(),
      };
    },
  },
  watch: {
    visible: function (val) {
      this.dialogVisible = val;
    },
    dialogVisible: function (val) {
      if (!val) {
        this.emitClose();
      }
    },
  },
};
</script>

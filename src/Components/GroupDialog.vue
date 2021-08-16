<template>
  <v-dialog id="group-dialog" v-model="dialogVisible" max-width="600px">
    <v-card>
      <v-card-title>
        {{ mode == 0 ? " Join Group" : "Create Group" }}
      </v-card-title>
      <v-card-text>
        <v-text-field
          label="Title"
          v-model="title"
          :rules="[titleRules]"
          counter
          maxlength="75"
        ></v-text-field>
        <v-text-field
          label="Password"
          v-model="password"
          type="password"
          counter
          maxlength="50"
        ></v-text-field>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn @click="emitClose" color="red">Cancel</v-btn>
        <v-btn v-if="mode == 0" @click="emitJoinGroup" color="primary"
          >Join</v-btn
        >
        <v-btn v-else @click="emitCreateGroup" color="primary">Create</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: {
    visible: Boolean,
    mode: { type: Number, required: true },
    initialTitle: { type: String, default: "" },
    initialPassword: { type: String, default: "" },
  },
  data() {
    return {
      dialogVisible: this.initialVisible,
      title: this.initialTitle,
      password: this.initialPassword,
    };
  },
  computed: {
    rules() {
      return !!this.title && !!this.password;
    },
  },
  methods: {
    emitCreateGroup() {
      this.$emit("create-group", {
        title: this.title,
        password: this.password,
      });
    },
    emitJoinGroup() {
      this.$emit("join-group", { title: this.title, password: this.password });
    },
    emitClose() {
      this.$emit("close");
    },
    titleRules(v) {
      return !!v || "Title Required";
    },
    setVisible(bool) {
      this.dialogVisible = bool;
    },
    reset() {
      this.title = "";
      this.password = "";
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

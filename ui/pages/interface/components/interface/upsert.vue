<template>
  <div id="upsert-section">
    <section>
      <b-tabs v-model="activeTab" :animated="false" :multiline="true">
        <template>
          <b-tab-item label="Settings">
            <div class="columns">
              <div class="column is-2">
                <div class="field">
                  <label class="label">Interface name</label>
                  <div class="control">
                    <input
                      class="input"
                      type="text"
                      placeholder="Interface name"
                      v-model="interfaceModel.interfaceName"
                      pattern="/\w+/"
                      @change="interfaceModel.interfaceName=$normalizeName(interfaceModel.interfaceName)"
                    />
                  </div>
                  <p class="help">unique name . use only alphanumeric letters</p>
                </div>
              </div>

              <div class="column is-10">
                <div class="field">
                  <label class="label">Tags</label>
                  <div class="control">
                    <input
                      class="input"
                      type="text"
                      placeholder="tags/keywords"
                      v-model="interfaceModel.tags"
                    />
                  </div>
                </div>
              </div>
            </div>

            <div>
              <div class="checkboxes">
                <b-checkbox v-model="interfaceModel.active">Is interface active ?</b-checkbox>
                <b-checkbox v-model="interfaceModel.listingAPI">Provide Listing API</b-checkbox>
              </div>

              <div class="field">
                <div class="label">Interface resuls type</div>
                <b-radio v-model="interfaceModel.resultType" native-value="all">All entries</b-radio>
                <b-radio v-model="interfaceModel.resultType" native-value="changes">Only changes</b-radio>
                <b-radio v-model="interfaceModel.resultType" native-value="new">Only new entries</b-radio>
                <b-radio
                  v-model="interfaceModel.resultType"
                  native-value="delete"
                >Only deleted entries</b-radio>
              </div>
              <div class="columns">
                <div class="column is-7">
                  <b-field label="Description">
                    <b-input maxlength="300" type="textarea" v-model="interfaceModel.description"></b-input>
                  </b-field>
                </div>
              </div>
            </div>
          </b-tab-item>
          <b-tab-item label="Source data">
            <data-source v-model="interfaceModel.source" :active="activeTab"></data-source>
          </b-tab-item>
          <b-tab-item label="Interface data props">
            <properties
              v-model="interfaceModel.properties"
              :source="interfaceModel.source"
              :active="activeTab"
            ></properties>
          </b-tab-item>
          <b-tab-item label="Output">
            <product-output v-model="interfaceModel.output" :active="activeTab"></product-output>
          </b-tab-item>
          <b-tab-item label="Notifications">
            <notifications v-model="interfaceModel.notifications" :active="activeTab"></notifications>
          </b-tab-item>
          <b-tab-item label="Job Scheduling">
            <job v-model="interfaceModel.job" :active="activeTab"></job>
          </b-tab-item>
          <b-tab-item label="Test">
            <test :interface="interfaceModel" :active="activeTab"></test>
          </b-tab-item>
          <b-tab-item label="Help">
            <help></help>
          </b-tab-item>
        </template>
      </b-tabs>
      <div class="buttons">
        <button class="button is-primary" v-show="!$route.query.id" @click="create(true)">Create</button>
        <button class="button is-link" v-show="$route.query.id" @click="update">Update</button>
      </div>
    </section>
    <h2 class="title is-4" v-show="!$route.query.id">Set new interface</h2>
    <h2 class="title is-4" v-show="$route.query.id">Update interface : {{$route.query.id}}</h2>
    {{origInterfaceName}}: {{interfaceModel}}
  </div>
</template>
<script>
import DataSource from "./dataSource.vue";
import Properties from "./properties.vue";
import ProductOutput from "./output.vue";
import Notifications from "./notifications.vue";

import Job from "./job.vue";
import Test from "./test.vue";

import Help from "./help.vue";

export default {
  name: "upsert",
  components: {
    DataSource,
    Properties,
    ProductOutput,
    Notifications,
    Job,
    Help,
    Test
  },
  data() {
    return {
      list: {},
      errors: [],
      activeTab: 0,
      origInterfaceName: null,
      interfaceModel: {
        interfaceName: "",
        description: "",
        source: {},
        properties: {},
        output: {},
        notifications: {},
        job: {},
        resultType: ""
      }
    };
  },
  watch: {
    activeTab: function() {
      // this.$router.push({ query: { activeTab: this.activeTab } });
      //   this.$set(this.$route.query, "activeTab", this.activeTab);
      this.$router.push({
        query: { ...this.$route.query, activeTab: this.activeTab }
      });
    }
  },
  async mounted() {
    if (this.$route.query.id) {
      Object.assign(
        this.interfaceModel,
        await this.$http.get("docs/interfaces/" + this.$route.query.id)
      );
      this.origInterfaceName = this.$route.query.id;
    }
    this.activeTab = this.$route.query.activeTab || 0;
  },
  methods: {
    validate() {
      this.errors = [];
      let hasError = false;
      if (!this.interfaceModel.interfaceName) {
        this.errors.push("missing interface name");
        hasError = true;
      }

      if (
        this.interfaceModel.interfaceName &&
        this.interfaceModel.interfaceName.match(/[\W|\s]/)
      ) {
        hasError = true;
        this.errors.push(
          "Invalid interface name  :use only [a-z] and _ letters"
        );
      }

      return hasError;
    },
    async update() {
      let self = this;
      if (self.validate()) return;
      self.interfaceModel._id = self.$route.query.id;
      let res1 = await this.$http.put("docs/interfaces", self.interfaceModel);
      self.interfaceModel._rev = res1.rev;
      if (self.origInterfaceName != self.interfaceModel.interfaceName) {
        let res = await self.$http.post("docs/rename", {
          db: "interfaces",
          id: res1.id,
          rev: res1.rev,
          targetId: self.interfaceModel.interfaceName
        });

        self.origInterfaceName = self.interfaceModel.interfaceName;
        self.interfaceModel._id = self.interfaceModel.interfaceName;
        self.interfaceModel._rev = res.rev;
        self.$route.query.id = self.interfaceModel.interfaceName;
      }
    },
    async create(list = true) {
      if (this.validate()) return;
      this.interfaceModel._id = this.interfaceModel.interfaceName;
      await this.$http.post("docs/interfaces", this.interfaceModel);
      this.$router.push({ name: "interfaces" });
    }
  },
  computed: {}
};
</script>
<style>
.b-radio.radio + .radio {
  margin-left: 50px;
}

.tabs li.is-active {
  border-bottom: 2px solid #0066ffa3;
}
.tabs li.is-active a {
  border: 0;
  font-weight: bolder;
  color: #000000c7;
}
</style>

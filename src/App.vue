<template>
  <div id="mainContent">
    <img alt="Vue logo" src="./assets/logo.png" />

    <section class="section">
      <div class="container">
        <div id="app">
          <div class="columns">
            <div class="column">
              <ContactGroup :store="db.contact_groups" :key="contactGroupCompKey" />
            </div>

            <div class="column is-three-quarters">
              <Contact :store="db.contacts" :key="contactCompKey" />
            </div>
          </div>

          <hr />
          <h2 class="title">Seeding Random Data</h2>
          <div class="columns">
            <div class="column">
              <b-field horizontal grouped label="Quantity">
                <b-input
                  v-model="seeding_quantity_for_each_model"
                  placeholder="Seeding quantity for each model"
                ></b-input>
                <p class="control">
                  <button class="button is-primary is-small" @click="seedData">
                    <span class="icon is-small">
                      <i class="fas fa-plus"></i>
                    </span>
                    <span>ADD RANDOM DATA</span>
                  </button>
                </p>
              </b-field>
            </div>
          </div>
        </div>
        <hr />

        <footer class="footer">
          <div class="content">
            <p>
              Made with
              <strong>IndexedDB</strong>,
              <strong>Vue</strong>,
              <strong>Bulma</strong>.
            </p>
          </div>
        </footer>
      </div>
    </section>
  </div>
</template>

<script>
import Dexie from "dexie";
import ContactGroup from "./components/ContactGroup.vue";
import Contact from "./components/Contact.vue";

/**
 * Initial DB
 */
const db = new Dexie("dexie");

db.version(1).stores({
  contact_groups: "++id, &name",
  contacts: "++id, firstName, lastName, &mobile, &email, groupId"
});

/**
 * Export default
 */
export default {
  name: "App",
  components: {
    ContactGroup,
    Contact
  },

  data() {
    return {
      db,
      seeding_quantity_for_each_model: null,
      contactGroupCompKey: 0,
      contactCompKey: 0
    };
  },

  methods: {
    /**
     * Get random int number between min and max
     */
    getRandomInt(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },

    /**
     * Force the components re-render
     */
    forceRerender() {
      this.contactGroupCompKey += 1;
      this.contactCompKey += 1;
    },

    /**
     * Seed data
     */
    seedData() {
      if (!this.seeding_quantity_for_each_model) {
        return false;
      }

      let contactGroupsList = [];
      let contactsList = [];
      for (let i = 0; i < this.seeding_quantity_for_each_model; i++) {
        contactGroupsList.push({
          name: this.$faker().company.companyName() + i
        });
        contactsList.push({
          firstName: this.$faker().name.firstName(),
          lastName: this.$faker().name.lastName(),
          mobile: this.$faker().phone.phoneNumber() + i,
          email: i + this.$faker().internet.email(),
          groupId: ""
        });
      }

      let self = this;
      db.transaction("rw", db.contact_groups, db.contacts, function() {
        // Bulk add contact groups
        db.contact_groups.bulkAdd(contactGroupsList);

        // Set random groupId from primary keys of table contact_groups
        db.contact_groups.toCollection().primaryKeys(contact_group_pks => {
          contactsList = contactsList.map(item => {
            item.groupId =
              contact_group_pks[
                self.getRandomInt(0, contact_group_pks.length - 1)
              ];

            return item;
          });

          // Bulk add contacts
          db.contacts.bulkAdd(contactsList);
        });
      }).catch(function(error) {
        // Log or display the error
        console.error(error.stack || error);
      });

      // Force the components re-render
      this.forceRerender();

      // Reset seeding quantity for each model
      this.seeding_quantity_for_each_model = null;
    }
  }
};
</script>

<style>
@import url("https://cdn.materialdesignicons.com/5.3.45/css/materialdesignicons.min.css");
@import url("https://use.fontawesome.com/releases/v5.2.0/css/all.css");

#mainContent {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#app {
  margin-bottom: calc(1.5rem - 0.75rem);
}

.mb-20 {
  margin-bottom: 20px;
}
</style>

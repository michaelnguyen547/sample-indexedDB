<template>
  <div>
    <a
      class="button is-primary modal-button is-small"
      @click="isModalActive = true, isInsertState = true, group = {}"
    >
      <span class="icon is-small">
        <i class="fas fa-plus"></i>
      </span>
      <span>ADD CONTACT GROUP</span>
    </a>

    <br />
    <br />

    <div class="card">
      <header class="card-header">
        <p class="card-header-title">Contact Group</p>
      </header>
      <div class="card-content">
        <div class="content">
          <table class="table" v-if="contact_groups.length">
            <thead>
              <tr>
                <th>Contact Group Name</th>
                <th>Action</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, index) in contact_groups" :key="index">
                <td v-text="row.name"></td>
                <td>
                  <a
                    title="edit"
                    class="modal-button"
                    @click="isModalActive = true, isInsertState = false, group = row"
                  >
                    <span class="icon is-small">
                      <i class="far fa-edit"></i>
                    </span>
                  </a>
                  <a title="delete" @click="confirmDelete(row.id)">
                    <span class="icon has-text-danger">
                      <i class="fas fa-ban"></i>
                    </span>
                  </a>
                </td>
              </tr>
            </tbody>
          </table>
          <p v-if="!contact_groups.length">No contact groups are found!</p>
        </div>
        <b-pagination
          v-if="contact_groups.length"
          :total="totalItems"
          :current.sync="currentPage"
          :range-before="rangeBefore"
          :range-after="rangeAfter"
          :order="order"
          :size="size"
          :simple="isSimple"
          :rounded="isRounded"
          :per-page="perPage"
          :icon-prev="prevIcon"
          :icon-next="nextIcon"
          aria-next-label="Next page"
          aria-previous-label="Previous page"
          aria-page-label="Page"
          aria-current-label="Current page"
          @change="onPageChanged"
        ></b-pagination>
      </div>
    </div>

    <b-modal :active.sync="isModalActive" has-modal-card>
      <div class="modal-card">
        <ValidationObserver ref="observer" v-slot="{ passes }">
          <form @submit.prevent="passes(submit)">
            <header class="modal-card-head">
              <p class="modal-card-title">Contact Group</p>
            </header>

            <section class="modal-card-body">
              <div class="field">
                <label class="label">Name</label>
                <ValidationProvider name="name" rules="required|min:4|max:15" v-slot="{ errors }">
                  <div class="control is-clearfix">
                    <input type="text" class="input" v-model="group.name" />
                  </div>
                  <p class="help is-danger" v-text="errors[0]"></p>
                </ValidationProvider>
              </div>
            </section>

            <footer class="modal-card-foot">
              <button type="submit" class="button is-success is-outlined is-small">
                <span>Save changes</span>
              </button>
            </footer>
          </form>
        </ValidationObserver>
      </div>
    </b-modal>
  </div>
</template>

<script>
export default {
  name: "ContactGroup",

  props: {
    store: Object
  },

  data() {
    return {
      group: {},
      contact_groups: [],
      isModalActive: false,
      isInsertState: undefined,
      totalItems: 0,
      currentPage: 1,
      perPage: 5, // TODO: limit per page
      rangeBefore: 3,
      rangeAfter: 2,
      order: "is-centered",
      size: "is-small",
      isSimple: false,
      isRounded: false,
      prevIcon: "chevron-left",
      nextIcon: "chevron-right"
    };
  },

  watch: {
    /**
     * Watching changes of flag isModalActive
     */
    isModalActive(val) {
      if (val) {
        this.$nextTick(() => {
          this.$refs.observer.reset();
        });
      }
    }
  },

  mounted() {
    this.refresh();
  },

  methods: {
    /**
     * Refresh list contact groups
     */
    refresh() {
      // Re-calculate total count
      this.countTotal();

      this.store
        .offset((this.currentPage - 1) * this.perPage)
        .limit(this.perPage)
        .toArray()
        .then(contact_groups => {
          this.contact_groups = contact_groups;
          // this.$root.$emit("refresh", contact_groups);
        });

      this.store.toArray().then(allContactGroups => {
        this.$root.$emit("refresh", allContactGroups);
      });
    },

    /**
     * Create new contact group
     */
    create() {
      this.store
        .add({
          name: this.group.name
        })
        .then(() => {
          this.refresh();
          this.isModalActive = false;
          this.$buefy.toast.open({
            message: "New contact group created!",
            type: "is-info"
          });
        })
        .catch(error => console.error(error));
    },

    /**
     * Update contact group
     */
    update() {
      this.store
        .put({
          id: this.group.id,
          name: this.group.name
        })
        .then(() => {
          this.refresh();
          this.isModalActive = false;
        })
        .catch(error => console.error(error));
    },

    /**
     * Show popup confirm delete contact group
     */
    confirmDelete(id) {
      this.$buefy.dialog.confirm({
        title: "Deleting group",
        message:
          "Are you sure you want to <b>delete</b> this group? This action cannot be undone.",
        confirmText: "Delete Contact Group",
        type: "is-danger",
        hasIcon: true,
        icon: "fa-exclamation-circle",
        iconPack: "fas",
        onConfirm: () => {
          this.store.delete(id).then(() => {
            this.$buefy.toast.open({
              message: "Contact Group deleted!",
              type: "is-info"
            });
            this.refresh();
          });
        }
      });
    },

    /**
     * Submit form create/update contact group
     */
    submit() {
      if (this.isInsertState) {
        this.create();
      } else {
        this.update();
      }
    },

    /**
     * Count total items
     */
    countTotal() {
      this.store.toArray().then(contact_groups => {
        this.totalItems = contact_groups.length;
      });
    },

    /**
     * Event on page changed
     */
    onPageChanged(pageNum) {
      this.currentPage = pageNum;
      this.refresh();
    }
  }
};
</script>

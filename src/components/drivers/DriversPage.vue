<template>
  <div class="drivers">
    <div class="drivers__sort sort-row">
      <div class="name">
        <span>name</span>
        <div class="triangles">
          <div class="triangle triangle--up"
               :class="{'triangle--active' : sortName === 'name' && sortType === 'asc'}"
               @click="sort('name', 'asc')">
          </div>
          <div class="triangle"
               :class="{'triangle--active' : sortName === 'name' && sortType === 'desc'}"
               @click="sort('name', 'desc')">
          </div>
        </div>
      </div>
      <div class="surname">
        <span>surname</span>
        <div class="triangles">
          <div class="triangle triangle--up"
               :class="{'triangle--active' : sortName === 'surname' && sortType === 'asc'}"
               @click="sort('surname', 'asc')">
          </div>
          <div class="triangle"
               :class="{'triangle--active' : sortName === 'surname' && sortType === 'desc'}"
               @click="sort('surname', 'desc')">
          </div>
        </div>
      </div>
      <div class="year">
        <span>year</span>
        <div class="triangles">
          <div class="triangle triangle--up"
               :class="{'triangle--active' : sortName === 'year' && sortType === 'asc'}"
               @click="sort('year', 'asc')">
          </div>
          <div class="triangle"
               :class="{'triangle--active' : sortName === 'year' && sortType === 'desc'}"
               @click="sort('year', 'desc')">
          </div>
        </div>
      </div>
      <div class="phone">
        <span>phone</span>
      </div>
    </div>
    <div class="drivers__table">
      <transition-group name="list">
        <driver-row v-for="driver in sortedDrivers"
                    :key="driver.id"
                    :driver="driver">
        </driver-row>
      </transition-group>
    </div>
  </div>
</template>

<script>
  import DriverRow from "./DriverRow"
  import {EventBus} from '../../utils/EventBus'

  export default {
    name: "DriversPage",
    components: {
      DriverRow
    },
    data() {
      return {
        drivers: [],
        sortedDrivers: [],
        sortName: '',
        sortType: ''
      }
    },
    created() {
      this.$emit('toggleLoader', true);

      this.apiRequest({url: "mmcba"}).then(
        response => {
          this.drivers = JSON.parse(response.data.body);
          this.sortedDrivers = this.drivers.slice();
          this.$emit('toggleLoader', false);
        },
        () => {
          console.log('connection error');
          this.$emit('toggleLoader', false);
        });

      EventBus.$on('createDriver', driver => this.createDriver(driver));
      EventBus.$on('updateDriver', driver => this.updateDriver(driver));
      EventBus.$on('deleteDriver', id => this.deleteDriver(id));
    },
    beforeDestroy() {
      EventBus.$off('createDriver');
      EventBus.$off('updateDriver');
      EventBus.$off('deleteDriver');
    },
    methods: {
      createDriver (driver) {
        driver.id = this.findMinUniqueId(this.drivers);
        this.drivers.unshift(driver);
        this.sort(this.sortName, this.sortType);
        this.putToServer("Driver has been created");
      },
      updateDriver(driver) {
        const index = this.drivers.findIndex(item => item.id === driver.id);

        this.drivers[index] = driver;
        this.sort(this.sortName, this.sortType);
        this.putToServer("Driver has been updated");
      },
      deleteDriver(id) {
        const index = this.drivers.findIndex(item => item.id === id);

        this.drivers.splice(index, 1);
        this.sort(this.sortName, this.sortType);
        this.putToServer("Driver has been deleted");
      },
      putToServer (message) {
        this.apiRequest({
          url: "mmcba",
          payload: JSON.stringify(this.drivers)
        }).then(
          () => EventBus.$emit('showNote', { message }),
          () => EventBus.$emit('showNote', { message: 'Network error' })
        );
      },
      sort(name, type) {
        if (this.sortName === name && this.sortType === type) {
          this.sortedDrivers = this.drivers.slice();
          this.sortName = '';
          this.sortType = '';
          return;
        }

        this.sortName = name;
        this.sortType = type;
        const rate = type === 'asc' ? 1 : -1;

        this.sortedDrivers = this.drivers.slice().sort((a, b) => a[name] > b[name] ? rate : rate * -1);
      }
    }
  }
</script>

<style lang="scss" scoped>
  .drivers {
    &__sort {
      display: flex;
      align-items: center;
      margin: 0 -16px;
      padding: 16px 0 16px 120px;
      background-color: #eee;

      .name {
        margin-right: 116px;
      }

      .surname {
        margin-right: 98px;
      }

      .year {
        margin-right: 38px;
      }
    }
  }
</style>

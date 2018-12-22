## Creating handy Notifications component
### The problem we solve
We want to notify user that some action happened. To do that we don't want to import an extra component to every component needed it or use store. Seems like we can resolve it at presentation layer.
### The idea
We will create one vue component with set of options. Then we will inject the function which creates an instance of this component and appends it to DOM tree.
### Implementation
First of all, you should create Notification component.
```javascript
// notification/Notification.vue
<template>
  <transition name="fade">
    <div
      v-show="visible"
      :class="wrapperClasses"
      @click="close"
    >
      <p>{{ message }}</p>
    </div>
  </transition>
</template>

<script>
export default {
  name: "Notification",
  data() {
    return {
      visible: false,
      position: "top-right",
      type: "",
      duration: 4500,
      timer: null,
      message: ""
    };
  },
  computed: {
    wrapperClasses() {
      return [
        'notification',
        `type-${this.type}`,
        { 'top-right': this.position === 'top-right' },
        { 'bottom-right': this.position === 'bottom-right' }
      ];
    }
  },
  methods: {
    close() {
      this.visible = false;
    }
  },
  mounted() {
    if (this.duration > 0) {
      this.timer = setTimeout(() => {
        this.close();
      }, this.duration);
    }
  }
};
</script>
```
Next we should in some way create an instance of this component and show it in UI. To do that we create functions which receive options, creates component and append it to DOM tree.
```javascript
// notification/index.js
import Vue from "vue";
import main from "./Notification";

// create a “subclass” of the base Vue constructor
const NotificationContructor = Vue.extend(main);

const Notification = function (options) {
  // create an instance of Notification and pass options to it
  const instance = new NotificationContructor({
    data: options
  });

  instance.vm = instance.$mount();
  document.body.appendChild(instance.vm.$el);
  instance.vm.visible = true;
  instance.dom = instance.vm.$el;

  return instance.vm;
};

export default Notification;
```
Last thing to do is inject this function to Vue instance to be able to call notification using **this**. As we use Nuxt.js, we can simply create new file **notification.js** in **plugins** folder.
```javascript
// plugins/notification.js
import Vue from 'vue'
import Notification from '@/components/notification'

// Setup
const Notifications = {
  install (Vue) {
    Vue.prototype.$notify = Notification
  }
}

Vue.use(Notifications)
```
Then we can simply call notification from any component using **this**.
```javascript
export default {
  methods: {
    async sendProposal () {
      try {
        await this.sendProposal()
        
        this.$notify({
          type: 'success',
          message: 'Proposal sent successfully'
        })
      } catch (err) {
        this.$notify({
          type: 'error',
          message: 'Error during fetching users'
        })
      }
    }
  }
}
```
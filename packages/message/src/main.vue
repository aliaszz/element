<template>
  <transition name="el-message-fade" @after-leave="handleAfterLeave">
    <div
      :class="[
        'el-message',
        type && !iconClass ? `el-message--${ type }` : '',
        center ? 'is-center' : '',
        showClose ? 'is-closable' : '',
        customClass
      ]"
      v-show="visible"
      @mouseenter="clearTimer"
      @mouseleave="startTimer"
      role="alert">
      <div class="el-message__button">
        <p v-if="cancelButtonText" class="el-message__cancel"
           @click="handleAction(cancelCallback)">{{ cancelButtonText }} ({{ time }})</p>
        <p v-if="confirmButtonText && cancelButtonText" class="el-message__separator">|</p>
        <p v-if="confirmButtonText" class="el-message__confirm"
           @click="handleAction(confirmCallback)">{{ confirmButtonText }}</p>
      </div>
      <slot>
        <p v-if="!dangerouslyUseHTMLString" class="el-message__content">{{ message }}</p>
        <p v-else v-html="message" class="el-message__content"></p>
      </slot>
      <i v-if="showClose" class="el-message__closeBtn el-icon-close" @click="close"></i>
    </div>
  </transition>
</template>

<script type="text/babel">
  const typeMap = {
    success: 'success',
    info: 'info',
    warning: 'warning',
    error: 'error',
    box: 'box'
  };

  export default {
    data() {
      return {
        visible: false,
        message: '',
        duration: 3000,
        time: 0,
        boxTimer: null,
        type: 'info',
        iconClass: '',
        customClass: '',
        onClose: null,
        showClose: false,
        closed: false,
        timer: null,
        dangerouslyUseHTMLString: false,
        center: false,
        confirmButtonText: '',
        cancelButtonText: '',
        confirmCallback: null,
        cancelCallback: null
      };
    },

    computed: {
      typeClass() {
        return this.type && !this.iconClass
          ? `el-message__icon el-icon-${ typeMap[this.type] }`
          : '';
      }
    },

    watch: {
      closed(newVal) {
        if (newVal) {
          this.visible = false;
        }
      }
    },

    methods: {
      handleAfterLeave() {
        this.$destroy(true);
        this.$el.parentNode.removeChild(this.$el);
      },

      close() {
        this.closed = true;
        if (typeof this.onClose === 'function') {
          this.onClose(this);
        }
      },

      clearTimer() {
        clearTimeout(this.timer);
      },

      startTimer() {
        if (this.duration > 0) {
          this.timer = setTimeout(() => {
            if (!this.closed) {
              this.close();
            }
          }, this.duration);
        }
      },
      keydown(e) {
        if (e.keyCode === 27) { // esc关闭消息
          if (!this.closed) {
            this.close();
          }
        }
      },
      handleAction(action) {
        if (!this.closed) {
          if (typeof action === 'function') {
            action();
          }
          this.close();
        }
      }
    },
    mounted() {
      this.startTimer();
      document.addEventListener('keydown', this.keydown);
      this.time = this.duration / 1000;
      if (this.time >= 30) {
        if (!this.boxTimer) {
          this.boxTimer = setInterval(() => {
            if (this.time > 0) {
              this.time--;
            } else {
              clearInterval(this.boxTimer);
              this.boxTimer = null;
            }
          }, 1000);
        }
      }
    },
    beforeDestroy() {
      document.removeEventListener('keydown', this.keydown);
    }
  };
</script>

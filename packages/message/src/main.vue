<template>
  <transition name="el-message-fade" @after-leave="handleAfterLeave">
    <div
      :class="[
        'el-message',
        type && !iconClass ? `el-message--${ type }` : '',
        center ? 'is-center' : '',
        showClose ? 'is-closable' : '',
        size ? `el-message__${size}`: '',
        customClass
      ]"
      :style="positionStyle"
      v-show="visible"
      @mouseenter="clearTimer"
      @mouseleave="startTimer"
      role="alert">
      <i :class="iconClass" v-if="iconClass"></i>
      <i :class="typeClass" v-else></i>
      <slot>
        <p v-if="title" class="el-message__title">{{ title }}</p>
        <p v-if="!dangerouslyUseHTMLString" class="el-message__content">{{ message }}</p>
        <p v-else v-html="message" class="el-message__content"></p>
      </slot>
      <i v-if="showClose" class="el-message__closeBtn el-icon-close" @click="close"></i>
      <div class="label-coutdown" :style="{visibility: !showClose && isTimerActive ? 'visible' : 'hidden'}">{{countDownSecond}}s</div>
    </div>
  </transition>
</template>

<script type="text/babel">
  const typeMap = {
    success: 'success',
    info: 'info',
    warning: 'warning',
    error: 'error'
  };

  export default {
    data() {
      return {
        visible: false,
        title: '',
        message: '',
        duration: 3000,
        type: 'info',
        size: '',
        iconClass: '',
        customClass: '',
        onClose: null,
        showClose: false,
        closed: false,
        verticalOffset: 20,
        timer: null,
        countdownTimer: null,
        isTimerActive: false,
        dangerouslyUseHTMLString: false,
        center: false,
        countDownSecond: 0
      };
    },

    computed: {
      typeClass() {
        return this.type && !this.iconClass
          ? `el-message__icon el-icon-${ typeMap[this.type] }`
          : '';
      },
      positionStyle() {
        return {
          'top': `${ this.verticalOffset }px`
        };
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
        this.isTimerActive = false;
        clearTimeout(this.timer);
        clearTimeout(this.countdownTimer);
        this.countDownSecond = Math.round(this.duration / 1000);
      },

      startTimer() {
        this.isTimerActive = true;
        if (this.duration > 0) {
          this.timer = setTimeout(() => {
            if (!this.closed) {
              this.close();
            }
          }, this.duration);
          this.countDown();
        }
      },
      countDown() {
        this.countDownSecond = Math.round(this.duration / 1000);
        const d = () => {
          if (this.countDownSecond > 0) {
            this.countdownTimer = setTimeout(() => {
              this.countDownSecond--;
              d();
            }, 1000);
          }
        };
        d();
      },
      keydown(e) {
        if (e.keyCode === 27) { // esc关闭消息
          if (!this.closed) {
            this.close();
          }
        }
      }
    },
    mounted() {
      this.startTimer();
      document.addEventListener('keydown', this.keydown);
    },
    beforeDestroy() {
      document.removeEventListener('keydown', this.keydown);
    }
  };
</script>
